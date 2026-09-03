# 🪺 NestJS RESTful API Study Case — Contact Management

Study notes for building the **Contact Management** REST API with **NestJS 11**, **Prisma 7**, **PostgreSQL**, **Zod 4**, and **Winston**. The app is a three-resource API — `User` → `Contact` → `Address` — secured by a token stored on the user row, validated by Zod schemas at the service layer, and covered end-to-end by **66 Jest + Supertest tests** across 3 suites.

These notes record the whole build: the architecture that emerged, every layer's job, and — most usefully — the **10 bugs** hit along the way, each with its symptom, root cause, fix, and the lesson it taught.

---

## 📋 Table of Contents

- [Requirement](#-requirement)
  - [Scaffold the Project](#scaffold-the-project)
  - [Install Dependencies](#install-dependencies)
  - [Setup Database](#setup-database)
  - [Run & Test](#run--test)
- [Architecture Overview](#-architecture-overview)
  - [Layered Structure](#layered-structure)
  - [Module Graph](#module-graph)
  - [Request Lifecycle](#request-lifecycle)
  - [The WebResponse Envelope](#the-webresponse-envelope)
- [Database Schema](#-database-schema)
- [Common Layer](#-common-layer)
  - [PrismaService](#prismaservice)
  - [ValidationService](#validationservice)
  - [ErrorFilter](#errorfilter)
  - [Authentication](#authentication)
- [User Module](#-user-module)
- [Contact Module](#-contact-module)
- [Address Module](#-address-module)
- [Testing Strategy](#-testing-strategy)
- [Bugs Encountered](#-bugs-encountered)
  - [1. Route mismatch — singular vs plural](#1-route-mismatch--singular-vs-plural)
  - [2. Destructured parameter without a default](#2-destructured-parameter-without-a-default)
  - [3. Prisma CLI crash — ERR_REQUIRE_ESM](#3-prisma-cli-crash--err_require_esm)
  - [4. Module exists but was never registered](#4-module-exists-but-was-never-registered)
  - [5. Foreign key violation in test setup](#5-foreign-key-violation-in-test-setup)
  - [6. Mutating the @Body() parameter](#6-mutating-the-body-parameter)
  - [7. Prisma update() throws instead of returning null](#7-prisma-update-throws-instead-of-returning-null)
  - [8. Validation runs before existence checks](#8-validation-runs-before-existence-checks)
  - [9. Winston — Unknown logger level](#9-winston--unknown-logger-level)
  - [10. Build output path vs start:prod](#10-build-output-path-vs-startprod)
- [Known Remaining Issues](#-known-remaining-issues)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)
- [Key Takeaways](#-key-takeaways)

---

## 🎯 Requirement

### Scaffold the Project

```bash
npx nest new study-nestjs-restful-api
# ✔ Which package manager would you ❤️ to use? npm
cd study-nestjs-restful-api
```

The generated `src/app.controller.ts`, `src/app.service.ts`, and the sample specs are deleted — this project is built from modules, not from the starter's single controller.

### Install Dependencies

```bash
# Validation
npm install zod

# Logging
npm install nest-winston winston

# Password hashing
npm install bcrypt
npm install --save-dev @types/bcrypt

# Config (.env support)
npm install @nestjs/config

# Database — Prisma 7 + the Postgres driver adapter
npm install --save-dev prisma@7
npm install @prisma/client@7 @prisma/adapter-pg
```

> **Key Insight:** Prisma **7** requires a *driver adapter* — the query engine no longer ships as a native binary that talks to Postgres directly. That is why `@prisma/adapter-pg` is a runtime dependency, and why `PrismaService` constructs `new PrismaPg({ connectionString })` instead of relying on the `datasource` URL alone.

### Setup Database

**`prisma/schema.prisma`** (generator + datasource)

```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}
```

Two Prisma 7 details worth noting:

- `provider = "prisma-client"` is the **new** generator (the old one was `prisma-client-js`). It emits ESM-friendly TypeScript into `output`, so the client is imported from `generated/prisma/client` — not from `@prisma/client`.
- The `datasource` block has **no `url`**. The connection string is supplied by `prisma7.config.ts` at CLI time and by `ConfigService` at runtime.

**`prisma7.config.ts`**

```typescript
import "dotenv/config";
import { defineConfig } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: { path: "prisma/migrations" },
  datasource: { url: process.env["DATABASE_URL"] },
});
```

**`.env`**

```bash
DATABASE_URL="postgres://postgres:<password>@localhost:5432/study_nestjs_restful_api"
```

Then run the migrations:

```bash
npx prisma migrate dev
```

Five migrations were produced over the course of the build:

| Migration | Purpose |
| --- | --- |
| `create_users_table` | `users` table, `username` as primary key |
| `create_contact_table` | `contacts` + FK to `users.username` |
| `create_address_table` | `addresses` + FK to `contacts.id` |
| `users_token_unique` | unique index on `users.token` (token lookup must be unambiguous) |
| `postal_code_must_exists` | made `addresses.postal_code` `NOT NULL` |

### Run & Test

```bash
npm run start:dev      # watch mode
npm run start:prod     # node dist/src/main   ← see Bug #10
npm test               # 66 tests, 3 suites
```

> ⚠️ **Note:** `npm test` is defined as `NODE_OPTIONS=--experimental-vm-modules jest --runInBand`. `--runInBand` is **required**, not an optimization: every suite truncates and reseeds the *same* `user-test` rows, so parallel workers would delete each other's fixtures mid-test.

---

## 🧱 Architecture Overview

### Layered Structure

Each feature module is the same four files, and each file has exactly one job:

```
src/<feature>/
├── <feature>.controller.ts   HTTP only — routes, params, status codes, response envelope
├── <feature>.service.ts      business logic — validate, authorize, query, map to response
├── <feature>.validation.ts   Zod schemas as static readonly members
└── <feature>.module.ts       wiring — providers, controllers, imports/exports
```

Plus two shared folders:

```
src/model/     request/response DTO classes (plain classes, no decorators)
src/common/    PrismaService, ValidationService, ErrorFilter, AuthMiddleware, @Auth
```

> **Key Insight:** Validation lives in the **service**, not in a `ValidationPipe` on the controller. The service calls `this.validationService.validate(Schema, request)` as its first statement. This keeps the schema reusable across callers and means the service can never be invoked with unvalidated input — but it also fixes the ordering that [Bug #8](#8-validation-runs-before-existence-checks) is about.

### Module Graph

**`src/app.module.ts`**

```typescript
@Module({
  imports: [CommonModule, UserModule, ContactModule, AddressModule],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

- `CommonModule` is `@Global()` — `PrismaService`, `ValidationService`, and `WinstonModule` are injectable everywhere without re-importing.
- `ContactModule` **exports** `ContactService` so `AddressModule` can import it and reuse `checkContactMustExist()`. This is the only cross-feature dependency in the app.

```
AppModule
├── CommonModule (@Global) ──> PrismaService, ValidationService, WinstonModule, ErrorFilter, AuthMiddleware
├── UserModule
├── ContactModule ──────────> exports ContactService
└── AddressModule ──────────> imports ContactModule
```

### Request Lifecycle

```
HTTP Request
   │
   ▼
AuthMiddleware            reads `Authorization` header → looks up user by token → sets req.user
   │                      (applied to '/api/*' — never blocks, just attaches)
   ▼
@Auth() param decorator   returns req.user, or throws UnauthorizedException (401)
   │
   ▼
Controller                parses params (ParseIntPipe), builds the service request object
   │
   ▼
Service                   validate (Zod) → authorize (must-exist checks) → Prisma query → map to *Response
   │
   ▼
ErrorFilter               catches HttpException / ZodError → { "errors": ... }
   │
   ▼
HTTP Response             { "data": ... } on success
```

### The WebResponse Envelope

**`src/model/web.model.ts`**

```typescript
export class WebResponse<T> {
  data: T;
  errors?: string;
  paging?: Paging;
}

export class Paging {
  current_page: number;
  size: number;
  total_page: number;
}
```

Every successful response is wrapped in `{ "data": ... }`; every failure is `{ "errors": ... }`. Controllers do the wrapping themselves (`return { data: result }`) — there is no global interceptor.

| Direction | Contract |
| --- | --- |
| Client → Server | `Authorization: <token>` — a raw UUID, **no `Bearer ` prefix** |
| Server → Client (success) | `{ "data": ... }`, plus `"paging"` on search |
| Server → Client (failure) | `{ "errors": ... }` |

> **Key Insight:** `ContactService.search()` is the one service that returns a full `WebResponse<ContactResponse[]>` itself rather than a bare payload — because it owns the `paging` metadata. Its controller therefore does `return result;` instead of `return { data: result };`. Every other endpoint wraps in the controller.

---

## 🗄️ Database Schema

**`prisma/schema.prisma`** (models)

```prisma
model User {
  username String   @id @db.VarChar(100)
  password String   @db.VarChar(100)
  name     String   @db.VarChar(100)
  token    String?  @unique @db.VarChar(100)

  contacts Contact[]

  @@map("users")
}

model Contact {
  id         Int      @default(autoincrement()) @id @db.Integer
  first_name String   @db.VarChar(100)
  last_name  String?  @db.VarChar(100)
  email      String?  @db.VarChar(100)
  phone      String?  @db.VarChar(25)
  username   String   @db.VarChar(100)

  user      User      @relation(fields: [username], references: [username])
  addresses Address[]

  @@map("contacts")
}

model Address {
  id          Int     @default(autoincrement()) @id @db.Integer
  street      String? @db.VarChar(255)
  city        String? @db.VarChar(100)
  province    String? @db.VarChar(100)
  postal_code String  @db.VarChar(10)
  country     String  @db.VarChar(100)
  contact_id  Int     @db.Integer

  contact Contact @relation(fields: [contact_id], references: [id])

  @@map("addresses")
}
```

Design decisions:

- **`username` is the primary key**, not a surrogate `id`. Contacts therefore key off `username`, and every ownership check is `where: { id, username }` — no join needed to prove a contact belongs to the caller.
- **`token` is nullable and unique.** Nullable because logout sets it to `null`; unique because `AuthMiddleware` looks a user up *by token*, so a duplicate would make the lookup ambiguous.
- **`@@map`** keeps DB tables snake-case plural while the Prisma models stay PascalCase singular.
- **`country` and `postal_code` are required** on `Address`; `street`, `city`, and `province` are optional. This asymmetry is enforced twice — once in the schema, once in `AddressValidation`.

---

## 🔐 Common Layer

### PrismaService

**`src/common/prisma.service.ts`**

```typescript
type PrismaServiceOptions = {
  adapter: PrismaPg;
  log: [
    { emit: 'event'; level: 'query' },
    { emit: 'event'; level: 'error' },
    { emit: 'event'; level: 'warn' },
    { emit: 'event'; level: 'info' },
  ];
};

@Injectable()
export class PrismaService
  extends PrismaClient<PrismaServiceOptions>
  implements OnModuleInit, OnModuleDestroy
{
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private readonly logger: Logger,
    config: ConfigService,
  ) {
    super({
      adapter: new PrismaPg({
        connectionString: config.getOrThrow<string>('DATABASE_URL'),
      }),
      log: [
        { emit: 'event', level: 'query' },
        /* ... error, warn, info ... */
      ],
    });
  }

  async onModuleInit() {
    this.$on('query', (e: Prisma.QueryEvent) => {
      this.logger.info('Prisma Query', {
        query: e.query, params: e.params, duration: e.duration,
      });
    });
    /* ... $on('error' | 'warn' | 'info') ... */
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Three things are happening here:

1. **`extends PrismaClient`** — the service *is* the client, so services call `this.prisma.contact.findMany(...)` directly.
2. **Lifecycle hooks** — `OnModuleInit` connects, `OnModuleDestroy` disconnects. Without the latter, Jest hangs on open handles between suites.
3. **Query logging into Winston** — every SQL statement, its params, and its duration are logged, which is what made [Bug #5](#5-foreign-key-violation-in-test-setup) diagnosable from the test output alone.

> **Key Insight:** The `PrismaServiceOptions` type alias exists purely so TypeScript can infer which event names `$on` accepts. The `log` tuple must stay a **literal tuple type** — widen it to `object[]` and `$on('query', ...)` stops type-checking.

### ValidationService

**`src/common/validation.service.ts`**

```typescript
@Injectable()
export class ValidationService {
  validate<T>(schema: z.ZodType<T>, data: T): T {
    try {
      return schema.parse(data);
    } catch (error) {
      throw new BadRequestException(error);
    }
  }
}
```

A four-line service that converts a Zod failure into a Nest `BadRequestException` (HTTP 400).

### ErrorFilter

**`src/common/error.filter.ts`**

```typescript
@Catch(ZodError, HttpException)
export class ErrorFilter implements ExceptionFilter {
  catch(exception: ZodError | HttpException, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse<Response>();

    if (exception instanceof HttpException) {
      response.status(exception.getStatus()).json({
        errors: exception.getResponse(),
      });
    } else if (exception instanceof ZodError) {
      response.status(400).json({ errors: 'Validation Error' });
    } else {
      response.status(500).json({ errors: 'Internal Server Error' });
    }
  }
}
```

Registered globally via `APP_FILTER` in `CommonModule`.

> **Key Insight:** The `ZodError` branch is effectively **dead code**. `ValidationService` already wraps every `ZodError` in a `BadRequestException`, so by the time the filter runs, the exception is an `HttpException` and the first branch wins. The observable result is that a 400 body carries the *raw serialized Zod error* rather than the tidy `"Validation Error"` string — see [Known Remaining Issues](#-known-remaining-issues).

Verified response shapes:

```json
// 400 — POST /api/users with empty strings
{ "errors": { "name": "ZodError", "message": "[\n  {\n    \"code\": \"too_small\", \"path\": [\"username\"], ... }\n]" } }
```

```json
// 401 — no Authorization header
{ "errors": { "message": "Unauthorized", "error": "Unauthorized", "statusCode": 401 } }
```

```json
// 404 — thrown by the app: HttpException('Contact not found', 404)
{ "errors": "Contact not found" }
```

```json
// 404 — unknown route (generated by Nest, not the app)
{ "errors": { "message": "Cannot GET /api/contact", "error": "Not Found", "statusCode": 404 } }
```

> ⚠️ **Note:** `errors` is **not a stable type**. `HttpException(string, status)` puts a plain *string* there, while Nest's own exceptions and `BadRequestException(zodError)` put an *object*. A client has to handle both.

### Authentication

Authentication is split in two: middleware **attaches**, the decorator **enforces**.

**`src/common/auth.middleware.ts`**

```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private readonly prismaService: PrismaService) {}

  async use(req: Request, res: Response, next: NextFunction): Promise<void> {
    const token = req.headers['authorization'] as string;
    if (token) {
      const user = await this.prismaService.user.findFirst({
        where: { token: token },
      });
      if (user) {
        req.user = user;
      }
    }
    next();
  }
}
```

**`src/common/auth.decorator.ts`**

```typescript
export const Auth = createParamDecorator(
  (data: unknown, ctx: ExecutionContext): User => {
    const request = ctx.switchToHttp().getRequest<Request>();
    const user = request.user as User;
    if (user) {
      return user;
    } else {
      throw new UnauthorizedException('Unauthorized');
    }
  },
);
```

**`src/common/common.module.ts`** (middleware registration)

```typescript
export class CommonModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes('/api/*');
  }
}
```

**`src/common/express.d.ts`** — teaches TypeScript that `req.user` exists:

```typescript
declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}
```

> **Key Insight:** The middleware **never rejects** a request — it only attaches `req.user` when a valid token is present, then always calls `next()`. Enforcement is the `@Auth()` decorator's job. That separation is what lets `POST /api/users` and `POST /api/users/login` sit under the same `/api/*` middleware without needing an exclusion list: they simply don't use `@Auth()`.

---

## 👤 User Module

**Endpoints**

| Method | Endpoint | Auth | Success | Purpose |
| --- | --- | --- | --- | --- |
| `POST` | `/api/users` | — | **201** | Register |
| `POST` | `/api/users/login` | — | 200 | Login → issues token |
| `GET` | `/api/users/current` | ✅ | 200 | Current user |
| `PATCH` | `/api/users/current` | ✅ | 200 | Update name and/or password |
| `DELETE` | `/api/users/current` | ✅ | 200 | Logout → clears token |

> ⚠️ **Note:** Register returns **201**, not 200 — it is the only endpoint without an explicit `@HttpCode(...)`, so Nest's POST default applies. Every other write endpoint declares `@HttpCode(200)` / `@HttpCode(HttpStatus.OK)`. The test asserts `toBe(201)` accordingly.

**`src/user/user.validation.ts`**

```typescript
export class UserValidation {
  static readonly REGISTER: ZodType<RegisterUserRequest> = z.object({
    username: z.string().min(1).max(100),
    password: z.string().min(1).max(100),
    name: z.string().min(1).max(100),
  });

  static readonly LOGIN: ZodType<LoginUserRequest> = z.object({
    username: z.string().min(1).max(100),
    password: z.string().min(1).max(100),
  });

  static readonly UPDATE: ZodType<UpdateUserRequest> = z.object({
    name: z.string().min(1).max(100).optional(),
    password: z.string().min(1).max(100).optional(),
  });
}
```

**`src/user/user.service.ts`** (register — duplicate check + hash)

```typescript
const totalUserWithSameUsername = await this.prismaService.user.count({
  where: { username: registerRequest.username },
});

if (totalUserWithSameUsername !== 0) {
  throw new HttpException('Username already exists', 400);
}

registerRequest.password = await bcrypt.hash(registerRequest.password, 10);
```

**`src/user/user.service.ts`** (login — verify then issue a token)

```typescript
let user = await this.prismaService.user.findUnique({
  where: { username: loginRequest.username },
});

if (!user) {
  throw new HttpException('Username or password is not valid', 401);
}

const isPasswordValid = await bcrypt.compare(loginRequest.password, user.password);

if (!isPasswordValid) {
  throw new HttpException('Username or password is not valid', 401);
}

user = await this.prismaService.user.update({
  where: { username: user.username },
  data: { token: randomUUID() },
});
```

Two security details:

- The same message — `'Username or password is not valid'` — is returned for both "no such user" and "wrong password", so the response can't be used to enumerate valid usernames.
- The token is a `randomUUID()` written to the user row. Logout is therefore just `data: { token: null }`; there is no JWT and no separate session store.

> **Key Insight:** `UserService.update()` mutates the `user` object it was handed and then passes that whole object as `data`. It works because the injected `user` came from `AuthMiddleware` and is a complete row — but it means the update statement rewrites every column, not just the changed ones.

---

## 📇 Contact Module

**Endpoints** — all return 200.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/contacts` | Create |
| `GET` | `/api/contacts/:contactId` | Get one |
| `PUT` | `/api/contacts/:contactId` | Update |
| `DELETE` | `/api/contacts/:contactId` | Delete |
| `GET` | `/api/contacts?name=&email=&phone=&page=&size=` | Search + paginate |

**`src/contact/contact.service.ts`** (the reusable ownership guard)

```typescript
async checkContactMustExist(username: string, contactId: number): Promise<Contact> {
  const contact = await this.prisma.contact.findUnique({
    where: { id: contactId, username: username },
  });

  if (!contact) {
    throw new HttpException('Contact not found', 404);
  }

  return contact;
}
```

This single method is the module's authorization primitive — `get`, `update`, `remove` all call it, and `AddressService` imports `ContactService` to call it too. Because `username` is part of the `where`, "not found" and "not yours" collapse into the same 404, which is the desired behavior: a user must not be able to detect that someone else's contact ID exists.

**`src/contact/contact.service.ts`** (search — dynamic filters)

```typescript
const filters: Prisma.ContactWhereInput[] = [{ username: user.username }];

if (searchRequest.name) {
  filters.push({
    OR: [
      { first_name: { contains: searchRequest.name } },
      { last_name:  { contains: searchRequest.name } },
    ],
  });
}

if (searchRequest.email) {
  filters.push({ email: { contains: searchRequest.email } });
}

if (searchRequest.phone) {
  filters.push({ phone: { contains: searchRequest.phone } });
}

const skip = (searchRequest.page - 1) * searchRequest.size;

const contacts = await this.prisma.contact.findMany({
  where: { AND: filters },
  take: searchRequest.size,
  skip: skip,
});

const totalCount = await this.prisma.contact.count({ where: { AND: filters } });

return {
  data: contacts.map((contact) => this.toContactResponse(contact)),
  paging: {
    current_page: searchRequest.page,
    size: searchRequest.size,
    total_page: Math.ceil(totalCount / searchRequest.size),
  },
};
```

The pattern is worth remembering: **start the filter array with the ownership predicate**, then conditionally push optional criteria, then combine with `AND`. The username filter can never be forgotten because it is the array's initial value.

**`src/contact/contact.controller.ts`** (optional numeric query params)

```typescript
@Query('page', new ParseIntPipe({ optional: true })) page?: number,
@Query('size', new ParseIntPipe({ optional: true })) size?: number,
```

`ParseIntPipe({ optional: true })` passes `undefined` through instead of throwing on a missing param; the controller then applies `page || 1` and `size || 10`.

> **Key Insight:** Zod 4 moved format validators to the top level — the schema uses **`z.email()`**, not the Zod 3 style `z.string().email()`.

---

## 📍 Address Module

Addresses are a **nested resource**: every route is scoped by its parent contact.

**`src/address/address.controller.ts`**

```typescript
@Controller('/api/contacts/:contactId/addresses')
export class AddressController { /* ... */ }
```

**Endpoints** — all return 200.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/contacts/:contactId/addresses` | Create |
| `GET` | `/api/contacts/:contactId/addresses/:addressId` | Get one |
| `PUT` | `/api/contacts/:contactId/addresses/:addressId` | Update |
| `DELETE` | `/api/contacts/:contactId/addresses/:addressId` | Delete |
| `GET` | `/api/contacts/:contactId/addresses` | List all for the contact |

**`src/address/address.module.ts`** (reusing the parent's guard)

```typescript
@Module({
  imports: [ContactModule],       // ← for ContactService.checkContactMustExist
  controllers: [AddressController],
  providers: [AddressService],
})
export class AddressModule {}
```

**`src/address/address.service.ts`** (the address-level guard)

```typescript
checkAddressMustExist = async ({
  username, contactId, addressId,
}: {
  username: string; contactId: number; addressId: number;
}) => {
  const address = await this.prisma.address.findFirst({
    where: {
      id: addressId,
      contact_id: contactId,
      contact: { username: username },       // ← join up to the owner
    },
  });

  if (!address) {
    throw new HttpException('Address not found or not belongs to user', 404);
  }

  return address;
};
```

Every write path is therefore a **two-step authorization**:

```typescript
await this.contactService.checkContactMustExist(user.username, validateRequest.contact_id);
await this.checkAddressMustExist({
  username: user.username,
  contactId: validateRequest.contact_id,
  addressId: validateRequest.id,
});
```

The contact check runs first so that a bad `contactId` reports "Contact not found" rather than "Address not found" — the more accurate error for the client.

**Merging URL params into the request body** — the controller builds a new object rather than mutating `@Body()`:

```typescript
const response = await this.addressService.update(user, {
  ...request,
  contact_id: contactId,
  id: addressId,
});
```

This shape is deliberate; mutating the body directly is exactly what [Bug #6](#6-mutating-the-body-parameter) was.

---

## 🧪 Testing Strategy

66 tests, 3 suites, all end-to-end through the real HTTP layer via Supertest against a real Postgres database — no mocks.

```
test/
├── test.module.ts     provides TestService
├── test.service.ts    fixture helpers: create/delete user, contact, address + login
├── user.spec.ts       16 tests
├── contact.spec.ts    27 tests
└── address.spec.ts    26 tests
```

**`test/test.service.ts`** (fixture helper with layered defaults)

```typescript
@Injectable()
export class TestService {
  app: INestApplication<App>;

  readonly username = 'user-test';
  readonly password = 'password-test';
  readonly name = 'User Test';
  /* ... firstName, lastName, email, phone, country, city, province, street, postalCode ... */

  async createUser({
    username = this.username,
    password = this.password,
    name = this.name,
  }: {
    username?: string;
    password?: string;
    name?: string;
  } = {}) {                                    // ← the `= {}` from Bug #2
    await this.prisma.user.create({
      data: {
        username,
        password: bcrypt.hashSync(password, 10),
        name,
      },
    });
  }

  async login(
    username: string = this.username,
    password: string = this.password,
  ): Promise<string> {
    const response = await request(this.app.getHttpServer())
      .post('/api/users/login')
      .send({ username, password })
      .expect(200);

    return (response.body as WebResponse<UserResponse>).data.token!;
  }
}
```

**Standard suite skeleton**

```typescript
beforeEach(async () => {
  const moduleFixture: TestingModule = await Test.createTestingModule({
    imports: [AppModule, TestModule],
  }).compile();

  app = moduleFixture.createNestApplication();
  await app.init();

  logger = app.get(WINSTON_MODULE_PROVIDER);
  testService = app.get(TestService);
  testService.app = app;
});

afterEach(async () => {
  await testService.deleteAddress();
  await testService.deleteContact();
  await testService.deleteUser();
  await app.close();
});
```

Cleanup order matters: **child before parent** (address → contact → user), or the foreign keys reject the deletes.

Each endpoint gets the same battery of cases:

- rejected if token is not provided → 401
- rejected if token is invalid → 401
- rejected if request is invalid → 400
- rejected if the parent/target is not found → 404
- happy path → 200 with field-by-field assertions

---

## 🐛 Bugs Encountered

### 1. Route mismatch — singular vs plural

**Symptom:** Eight `GET /api/contacts` tests failed with `Expected: 200, Received: 404`, and later a manual `POST http://localhost:3000/api/contacts` returned `Cannot POST /api/contacts` — while `GET /api/users/current` worked fine.

**Code:**

```typescript
// ❌ src/contact/contact.controller.ts
@Controller('/api/contact')          // singular

// but the sibling modules were plural:
@Controller('/api/users')
@Controller('/api/contacts/:contactId/addresses')
```

**Root cause:** The contact controller was registered at `/api/contact` while the tests, the manual `.http` file, and the *nested address routes* all assumed `/api/contacts`. A Nest route that doesn't match simply 404s — there is no warning at startup.

**Fix:**

```typescript
// ✅
@Controller('/api/contacts')
```

**Lesson:** When a test and the code disagree, decide **which one is the source of truth** before editing either. The first attempt here rewrote the *tests* to singular to make them pass — which "worked" and hid the real defect. The evidence that the controller was wrong was already in the repo: `/api/users` was plural, and `AddressController` nested itself under `/api/contacts/:contactId/addresses`. Making the tests match a buggy route just moved the bug somewhere a test could no longer see it, and it resurfaced the moment a real HTTP client hit the API.

### 2. Destructured parameter without a default

**Symptom:** All 25 tests in a suite — then all 40 across two suites — failed at once with:

```
TypeError: Cannot read properties of undefined (reading 'username')
  at TestService.createUser (test.service.ts:26:5)
```

**Code:**

```typescript
// ❌
async createUser({
  username = this.username,
  password = this.password,
  name = this.name,
}: {
  username?: string;
  password?: string;
  name?: string;
}) {                          // ← no default for the parameter itself
```

Called everywhere as `await testService.createUser();`

**Root cause:** Refactoring the helpers from positional parameters to a destructured options object left off the default for the **whole object**. With no argument passed, the parameter is `undefined`, and destructuring `undefined` throws before any of the per-field defaults can apply. The `?` marks in the type annotation are compile-time only — they do nothing at runtime.

**Fix:**

```typescript
// ✅
}: {
  username?: string;
  password?: string;
  name?: string;
} = {}) {                     // ← default for the parameter itself
```

**Lesson:** An options object needs **two layers of defaults**: `= {}` for the parameter (so a zero-argument call gets an object instead of `undefined`), and `field = fallback` inside the pattern (so a missing key gets a value). Optional *types* are not optional *values*.

### 3. Prisma CLI crash — ERR_REQUIRE_ESM

**Symptom:** Every `prisma` command — even `npx prisma -v` — crashed before doing anything:

```
Error [ERR_REQUIRE_ESM]: require() of ES Module .../zeptomatch/dist/index.js
from .../@prisma/dev/dist/state.cjs not supported.
```

**Root cause:** `@prisma/dev` (a dependency of the `prisma` CLI, eagerly imported at startup regardless of subcommand) `require()`s `zeptomatch`, which is a pure-ESM package (`"type": "module"`). Checking the registry confirmed this was **not fixable in the project**: the newest `@prisma/dev` (0.25.2) exact-pins the same ESM-only `zeptomatch@2.1.0`, so bumping the dependency changes nothing.

The real variable was **Node**. Support for `require()`-ing an ESM module became default-on in **Node v22.12.0** (and v20.19.0 on the v20 line). The installed version was **v22.11.0** — one minor release short.

**Fix:**

```bash
nvm install 22.23.2
nvm alias default 22.23.2
```

```
$ npx prisma -v
prisma : 7.10.0
Node.js : v22.23.2      ← works
```

**Lesson:** Before hunting for a project-side workaround, check whether the failure is **environmental**. A stack trace pointing entirely into `node_modules` — with nothing of yours in it — is a strong signal that the runtime or a dependency version is the variable, not your code. Verifying that the *latest* version of the offending package still had the bug is what ruled out "just upgrade the library" and pointed at Node instead.

### 4. Module exists but was never registered

**Symptom:** Every address route returned 404, even though `address.controller.ts`, `address.service.ts`, and `address.module.ts` all existed and compiled.

**Code:**

```typescript
// ❌ src/app.module.ts
@Module({
  imports: [CommonModule, UserModule, ContactModule],   // AddressModule missing
})
```

**Root cause:** Writing a module does not register it. Nest only maps routes for controllers reachable from `AppModule`'s import graph, and an unimported module is silently inert — no error, no warning, just 404s.

**Fix:**

```typescript
// ✅
imports: [CommonModule, UserModule, ContactModule, AddressModule],
```

**Lesson:** In Nest, "the file exists" and "the route exists" are different claims. When a whole group of endpoints 404s while individual ones work, suspect **wiring**, not the handler — and check `app.module.ts` first.

### 5. Foreign key violation in test setup

**Symptom:**

```
PrismaClientKnownRequestError:
Invalid `this.prisma.contact.create()` invocation
Foreign key constraint violated on the constraint: `contacts_username_fkey`
```

**Code:**

```typescript
// ❌ test/address.spec.ts
beforeEach(async () => {
  await testService.deleteAddress();
  await testService.deleteContact();
  await testService.deleteUser();
  contactResponse = await testService.createContact();   // no user exists!
});
```

**Root cause:** The setup deleted the user and then created a contact without recreating the user. `contacts.username` is a foreign key to `users.username`, so the insert had nothing to point at.

**Fix:**

```typescript
// ✅
  await testService.deleteUser();
  await testService.createUser();          // ← restore the parent first
  contactResponse = await testService.createContact();
```

**Lesson:** Fixture setup has to respect the **same ordering the foreign keys impose**, mirrored: delete children → parents, then create parents → children. The Prisma query logging in `PrismaService` is what made this immediately readable — the failing `INSERT` and its params were right there in the test output.

### 6. Mutating the @Body() parameter

**Symptom:** Two "not found" tests expected 404 but got 500. The server log showed:

```
ERROR [ExceptionsHandler] TypeError: Cannot set properties of undefined (setting 'contact_id')
    at AddressController.update (src/address/address.controller.ts:64:23)
```

**Code:**

```typescript
// ❌ src/address/address.controller.ts
async update(
  @Auth() user: User,
  @Param('contactId', ParseIntPipe) contactId: number,
  @Param('addressId', ParseIntPipe) addressId: number,
  @Body() request: UpdateAddressRequest,
) {
  request.contact_id = contactId;      // request is undefined when no body is sent
  request.id = addressId;
  const response = await this.addressService.update(user, request);
```

**Root cause:** The tests called `.send()` with **no body**, so `@Body()` resolved to `undefined`. Assigning a property to `undefined` throws a `TypeError`, which isn't an `HttpException`, so `ErrorFilter` never handled it and Nest's default handler returned 500 — masking the 404 the test was actually checking for.

`ContactController` had already avoided this by **spreading into a new object** instead of mutating:

```typescript
// ✅ src/contact/contact.controller.ts — the pattern that was already correct
const result = await this.contactService.update(user, { ...request, id: contactId });
```

**Fix:**

```typescript
// ✅ src/address/address.controller.ts
const response = await this.addressService.update(user, {
  ...request,
  contact_id: contactId,
  id: addressId,
});
```

**Lesson:** `{ ...undefined }` is legal and yields `{}`; `undefined.foo = 1` throws. Building a **new object** is therefore both safer and cleaner than mutating an injected parameter — and when two controllers in the same codebase do the same job differently, the divergence itself is worth a second look.

### 7. Prisma update() throws instead of returning null

**Symptom:** The "address not found" update test returned 500 rather than 404.

**Code:**

```typescript
// ❌ src/address/address.service.ts — update() went straight to the write
await this.contactService.checkContactMustExist(user.username, validateRequest.contact_id);

const address = await this.prisma.address.update({          // throws P2025 if absent
  where: { id: validateRequest.id, contact_id: validateRequest.contact_id },
  data: { /* ... */ },
});
```

**Root cause:** `get()` did a `findUnique` and threw an explicit `HttpException(..., 404)` when the row was missing, but `update()` skipped that check and relied on `prisma.address.update()`. When no row matches, Prisma raises `P2025` — a `PrismaClientKnownRequestError`, not an `HttpException` — so it escaped `ErrorFilter` as a 500.

**Fix:** an explicit existence check before the write, matching `get()`:

```typescript
// ✅
await this.checkAddressMustExist({
  username: user.username,
  contactId: validateRequest.contact_id,
  addressId: validateRequest.id,
});

const address = await this.prisma.address.update({ /* ... */ });
```

**Lesson:** Prisma's write methods **throw** on a missing row while its read methods return `null`. Any endpoint that must answer 404 needs its own read-and-check step first — otherwise the ORM's error type decides your status code, and it will pick 500.

### 8. Validation runs before existence checks

**Symptom:** After fixing Bug #6 and #7, the same two tests moved from 500 to **400**, still not the expected 404.

**Root cause:** Not a code bug at all — an **ordering contract**. Every service validates first:

```typescript
const validateRequest = this.validationService.validate(AddressValidation.UPDATE, request);   // 1. validate → 400
await this.contactService.checkContactMustExist(...);                                          // 2. exists?  → 404
await this.checkAddressMustExist(...);                                                         // 3. exists?  → 404
```

The tests sent an **empty body** while `AddressValidation.UPDATE` requires `country` and `postal_code`. Validation failed first, so the request never reached the existence checks. `contact.spec.ts` documents this same behavior explicitly — its equivalent test sends `.send()` empty and asserts **400**.

**Fix:** send a valid body so the request survives validation and the 404 path is actually exercised:

```typescript
// ✅ test/address.spec.ts
.put(`/api/contacts/${contactResponse.id + 1}/addresses/${addressResponse.id}`)
.set('Authorization', token)
.send({
  country: testService.country,
  city: testService.city,
  province: testService.province,
  street: testService.street,
  postal_code: testService.postalCode,
});
```

**Lesson:** To test a specific failure, **every other input must be valid** — otherwise an earlier guard answers first and the test silently asserts the wrong thing. Write the test to isolate the one condition it names.

### 9. Winston — Unknown logger level

**Symptom:** Every test printed `console.error [winston] Unknown logger level: undefined`, burying the real output. Tests still passed.

**Code:**

```typescript
// ❌
logger.log(response.body);
```

**Root cause:** The `logger` is the raw Winston instance (`app.get(WINSTON_MODULE_PROVIDER)`), and Winston's `.log()` expects either `(level, message)` or an object carrying a `level` field. A bare response body has neither, so the level resolved to `undefined`.

**Fix:**

```typescript
// ✅
logger.log('info', response.body);
```

**Lesson:** `LoggerService` (Nest) and `Logger` (Winston) have **different `.log()` signatures** — Nest's takes `(message, ...context)`, Winston's takes `(level, message)`. The variable was *typed* as Nest's `LoggerService` but *held* a Winston logger, so TypeScript couldn't catch the mismatch. A type annotation that doesn't match the runtime object gives up exactly the protection you added it for.

### 10. Build output path vs start:prod

**Symptom:** `npm run build` succeeded, then `npm run start:prod` crashed:

```
Error: Cannot find module '/Users/.../dist/main'
code: 'MODULE_NOT_FOUND'
```

**Root cause:** The compiler emitted `dist/src/main.js`, not `dist/main.js`. `tsconfig.json` sets `outDir` but no `rootDir`, so TypeScript infers the root from the **common directory of all inputs** — and `prisma7.config.ts` sits at the project root. That widened the common root from `src/` to the project root, so the `src/` folder was preserved inside `dist/`. The leftover `dist/prisma7.config.js` is the fingerprint:

```
dist/prisma7.config.js      ← root-level input pulled the common root up
dist/src/main.js            ← so main.js landed one level deeper
```

**Fix:**

```json
// ✅ package.json
"start:prod": "node dist/src/main"
```

(Alternatively, pin `"rootDir": "./src"` and exclude the root-level config from the build to get `dist/main.js` back.)

**Lesson:** With `outDir` but no `rootDir`, the output layout is **derived from the input set** — adding one file at the project root silently relocates every emitted file. If a build "succeeds" but the entry point moved, list `dist/` before editing scripts.

---

## ⚠️ Known Remaining Issues

| Issue | Where | Detail |
| --- | --- | --- |
| Zod errors leak internals | `src/common/validation.service.ts` | A 400 body is `{"errors":{"name":"ZodError","message":"[...]"}}` — the `message` is a *JSON string* of the raw issue array. A field→message map would be far easier for a client to consume. |
| `errors` has no stable type | `src/common/error.filter.ts` | `errors` is a plain **string** for `HttpException('Contact not found', 404)` but an **object** for Nest's built-ins and for wrapped Zod errors. Clients must handle both shapes. |
| `ZodError` branch is dead code | `src/common/error.filter.ts` | `ValidationService` already wraps every `ZodError` in a `BadRequestException`, so the filter's `ZodError` branch (and its tidy `'Validation Error'` string) never runs. |
| `else` branch is unreachable | `src/common/error.filter.ts` | `@Catch(ZodError, HttpException)` only routes those two types here, so the `500 Internal Server Error` fallback can't fire. Non-`HttpException` errors (e.g. Prisma's `P2025`) bypass the filter entirely — which is what made Bug #7 a 500. |
| `update()` rewrites every column | `src/user/user.service.ts` | It mutates the injected `user` row and passes the whole object as `data`, so unchanged columns are rewritten too. |
| Passwords logged in cleartext | `src/user/user.service.ts` | `UserService.login()` logs the full request — `{"username":"...","password":"..."}` — at `info` level. Fine for a study project, unacceptable in production. |
| Search is case-sensitive | `src/contact/contact.service.ts` | `contains` without `mode: 'insensitive'` means `"John"` won't match `"john"` on Postgres. |
| No `.nvmrc` | project root | Node ≥ 22.12.0 is a hard requirement (Bug #3) but nothing in the repo records it. |
| Negative IDs answer 400, not 404 | `src/address/address.validation.ts` | `ParseIntPipe` accepts negatives, so `PUT /api/contacts/-1/addresses/-1` reaches Zod's `.positive()` and returns **400** — while `GET /api/contacts/-1` and the *list* route (no schema) correctly return **404**. Same missing resource, two different status codes depending on whether the route happens to validate its IDs. |

---

## 📚 Quick Reference

### All Endpoints

| Module | Method | Endpoint | Auth | Success |
| --- | --- | --- | --- | --- |
| User | `POST` | `/api/users` | — | 201 |
| User | `POST` | `/api/users/login` | — | 200 |
| User | `GET` | `/api/users/current` | ✅ | 200 |
| User | `PATCH` | `/api/users/current` | ✅ | 200 |
| User | `DELETE` | `/api/users/current` | ✅ | 200 |
| Contact | `POST` | `/api/contacts` | ✅ | 200 |
| Contact | `GET` | `/api/contacts/:contactId` | ✅ | 200 |
| Contact | `PUT` | `/api/contacts/:contactId` | ✅ | 200 |
| Contact | `DELETE` | `/api/contacts/:contactId` | ✅ | 200 |
| Contact | `GET` | `/api/contacts?name=&email=&phone=&page=&size=` | ✅ | 200 + `paging` |
| Address | `POST` | `/api/contacts/:contactId/addresses` | ✅ | 200 |
| Address | `GET` | `/api/contacts/:contactId/addresses/:addressId` | ✅ | 200 |
| Address | `PUT` | `/api/contacts/:contactId/addresses/:addressId` | ✅ | 200 |
| Address | `DELETE` | `/api/contacts/:contactId/addresses/:addressId` | ✅ | 200 |
| Address | `GET` | `/api/contacts/:contactId/addresses` | ✅ | 200 |

### Status Code Contract

| Code | Meaning | Raised by |
| --- | --- | --- |
| 200 | Success | `@HttpCode(200)` on every endpoint except register |
| 201 | Created | Nest's POST default — register only |
| 400 | Validation failed / username taken | `ValidationService` → `BadRequestException`; `HttpException('Username already exists', 400)` |
| 401 | No / invalid / expired token | `@Auth()` → `UnauthorizedException`; login failures |
| 404 | Not found **or** not yours | `checkContactMustExist`, `checkAddressMustExist` |

### Zod Validation Rules

| Schema | Required | Optional |
| --- | --- | --- |
| `UserValidation.REGISTER` | `username`, `password`, `name` — all 1–100 | — |
| `UserValidation.LOGIN` | `username`, `password` — 1–100 | — |
| `UserValidation.UPDATE` | — | `name`, `password` — 1–100 |
| `ContactValidation.CREATE` | `first_name` 1–100 | `last_name` 1–100, `email` (`z.email()`), `phone` 1–20 |
| `ContactValidation.UPDATE` | `id` positive, `first_name` | `last_name`, `email`, `phone` |
| `ContactValidation.SEARCH` | — | `name`, `email`, `phone`, `page` ≥1 (default 1), `size` ≥1 (default 10) |
| `AddressValidation.CREATE` | `contact_id` int+, `country` 1–100, `postal_code` 1–10 | `street`, `city`, `province` 1–100 |
| `AddressValidation.UPDATE` | `id`, `contact_id`, `country`, `postal_code` | `street`, `city`, `province` |
| `AddressValidation.GET` / `.REMOVE` | `contact_id`, `address_id` — int+ | — |

### Key Building Blocks

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| Global module | Share providers app-wide | `@Global() @Module({ exports: [...] })` |
| Global filter | Uniform error envelope | `{ provide: APP_FILTER, useClass: ErrorFilter }` |
| Middleware | Attach `req.user` from token | `consumer.apply(AuthMiddleware).forRoutes('/api/*')` |
| Param decorator | Inject + enforce the user | `createParamDecorator((data, ctx) => ...)` |
| Nested routes | Scope a child to its parent | `@Controller('/api/contacts/:contactId/addresses')` |
| Numeric params | Parse + validate path/query ints | `@Param('id', ParseIntPipe)`, `new ParseIntPipe({ optional: true })` |
| Lifecycle hooks | Connect / disconnect Prisma | `implements OnModuleInit, OnModuleDestroy` |
| Prisma driver adapter | Prisma 7 Postgres connection | `new PrismaPg({ connectionString })` |
| Dynamic filters | Compose optional search criteria | `Prisma.ContactWhereInput[]` + `where: { AND: filters }` |
| E2E bootstrap | Real app in tests | `Test.createTestingModule({ imports: [AppModule, TestModule] })` |

### npm Scripts

| Script | Command |
| --- | --- |
| `build` | `nest build` |
| `start:dev` | `nest start --watch` |
| `start:prod` | `node dist/src/main` |
| `test` | `NODE_OPTIONS=--experimental-vm-modules jest --runInBand` |
| `test:cov` | same, plus `--coverage` |
| `lint` | `eslint "{src,apps,libs,test}/**/*.ts" --fix` |
| `format` | `prettier --write "src/**/*.ts" "test/**/*.ts"` |

---

## 💡 Best Practices

**✅ Do This**

- Give an options-object parameter **both** layers of defaults — `= {}` on the parameter and `field = fallback` inside the pattern.
- Build a **new object** (`{ ...request, id }`) instead of mutating an injected `@Body()`; it survives a missing body and reads better.
- Put the ownership predicate **first** in a filter array so it can never be forgotten: `const filters = [{ username: user.username }]`.
- Collapse "not found" and "not yours" into a single **404** by putting `username` in the `where` clause — it prevents ID enumeration.
- Do an explicit **read-and-check** before any `update`/`delete` that must be able to answer 404.
- Extract must-exist guards into named methods (`checkContactMustExist`) and export the service so sibling modules can reuse them.
- Clean up test fixtures **child → parent**, and create them **parent → child**.
- Make each negative test vary **exactly one** thing; keep every other input valid.
- Register the query logger on `PrismaService` — the SQL and params in the test output are what turn FK errors into one-minute fixes.

**❌ Avoid This**

- Don't rewrite a **test** to match the code until you've decided which one is the source of truth — Bug #1 hid a real routing defect that way.
- Don't assume writing a module registers it; an unimported module 404s **silently**.
- Don't rely on `prisma.update()` to produce a 404 — it throws `P2025`, which becomes a 500.
- Don't annotate a variable with one logger type while assigning another (`LoggerService` vs Winston's `Logger`); the signatures differ and the mismatch is invisible to TypeScript.
- Don't log whole request bodies at `info` when they can contain passwords.
- Don't trust `outDir` alone to shape `dist/` — without `rootDir`, one root-level `.ts` file relocates the entry point.
- Don't run these suites in parallel; they share the `user-test` fixture rows, so `--runInBand` is mandatory.
- Don't reach for a project-side workaround before ruling out the **environment** — Bug #3 was a Node minor version, not a dependency to patch.

---

## 🎯 Key Takeaways

- **A 404 in Nest is usually a wiring question, not a handler question.** Both Bug #1 (wrong controller path) and Bug #4 (unregistered module) presented identically — a whole group of endpoints returning 404 with perfectly good code behind them.
- **The ORM's error type quietly picks your status code.** Prisma reads return `null`; Prisma writes throw. Bug #7's 500 was Prisma deciding the response because the service never checked first.
- **TypeScript's optionality is compile-time only.** Bug #2 (`?` in the type, no `= {}` at runtime) and Bug #9 (annotated as `LoggerService`, actually a Winston `Logger`) were both cases of the type system describing something the runtime didn't honor.
- **Order of guards is a public contract.** Validate-then-check means an invalid body yields 400 even when the resource is missing (Bug #8). Whatever order you pick, the tests must encode it.
- **Fix the source of truth, not the symptom.** The most instructive bug here was #1: making the tests match a buggy route turned a failing test into a passing one *and* a latent production 404. A test rewritten to accommodate a defect stops being a test.
- **Some bugs live outside the repo.** Bug #3 (Node < 22.12.0) and Bug #10 (`rootDir` inference) were environment and build-config problems that no amount of reading `src/` would have found.
