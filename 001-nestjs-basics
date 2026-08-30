# 🪺 NestJS Basics

A practical reference guide for learning NestJS from scratch — covering modules, controllers, providers, dependency injection, and the full request-handling pipeline (middleware, guards, interceptors, pipes, filters), built hands-on inside the `study-nestjs-basic` project.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is NestJS?](#what-is-nestjs)
  - [Why Use a Framework?](#why-use-a-framework)
  - [What NestJS Uses Under the Hood](#what-nestjs-uses-under-the-hood)
- [Project Setup](#-project-setup)
  - [Installing the NestJS CLI](#installing-the-nestjs-cli)
  - [Creating a Project](#creating-a-project)
  - [Folder Structure](#folder-structure)
  - [Running the Application](#running-the-application)
- [Decorators](#-decorators)
  - [What a Decorator Is](#what-a-decorator-is)
  - [Where Decorators Appear in NestJS](#where-decorators-appear-in-nestjs)
- [Modules](#-modules)
  - [The Application Module](#the-application-module)
  - [Creating a Module](#creating-a-module)
- [Controllers](#-controllers)
  - [Creating a Controller](#creating-a-controller)
  - [Routing & HTTP Method Decorators](#routing--http-method-decorators)
  - [Route Order Matters](#route-order-matters)
- [Handling the Request](#-handling-the-request)
  - [The Express Request Object](#the-express-request-object)
  - [Request Decorators](#request-decorators)
  - [Reading Query & Route Params](#reading-query--route-params)
- [Shaping the Response](#-shaping-the-response)
  - [The Express Response Object](#the-express-response-object)
  - [Response Decorators](#response-decorators)
  - [Asynchronous Handlers](#asynchronous-handlers)
- [Cookies](#-cookies)
- [Views & Template Engines](#-views--template-engines)
- [Testing](#-testing)
  - [Unit Testing](#unit-testing)
  - [Mocking Request & Response](#mocking-request--response)
  - [Integration (E2E) Testing](#integration-e2e-testing)
- [Providers & Dependency Injection](#-providers--dependency-injection)
  - [Creating a Provider](#creating-a-provider)
  - [Constructor Injection](#constructor-injection)
  - [Property-Based Injection](#property-based-injection)
  - [Optional Dependencies](#optional-dependencies)
- [Custom Providers](#-custom-providers)
  - [Standard Provider](#standard-provider)
  - [Class Provider](#class-provider)
  - [Value Provider](#value-provider)
  - [Factory Provider](#factory-provider)
  - [Alias Provider](#alias-provider)
- [Module Reference](#-module-reference)
- [Configuration](#-configuration)
- [Shared Modules](#-shared-modules)
- [Database with Prisma](#-database-with-prisma)
- [Logging with Winston](#-logging-with-winston)
- [Global Modules](#-global-modules)
- [Dynamic Modules](#-dynamic-modules)
- [Validation](#-validation)
- [Middleware](#-middleware)
- [Exception Filters](#-exception-filters)
  - [Creating an Exception Filter](#creating-an-exception-filter)
  - [HttpException](#httpexception)
  - [Global Exception Filters](#global-exception-filters)
- [Pipes](#-pipes)
  - [Built-in Pipes](#built-in-pipes)
  - [Creating a Pipe](#creating-a-pipe)
  - [Global Pipes](#global-pipes)
- [Interceptors](#-interceptors)
- [Custom Decorators](#-custom-decorators)
- [Guards](#-guards)
- [Lifecycle Events](#-lifecycle-events)
- [Reflector](#-reflector)
- [Global Providers](#-global-providers)
- [Request Lifecycle](#-request-lifecycle)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is NestJS?

- **Nest (NestJS)** is a framework for building efficient, server-side applications on top of Node.js
- It uses **TypeScript** as its primary language, but plain JavaScript works too
- It is one of the most popular frameworks in the TypeScript ecosystem
- Its architecture is heavily inspired by Angular — modules, decorators, and dependency injection are the three pillars you'll meet everywhere

> Reference: [nestjs.com](https://nestjs.com/) · [docs.nestjs.com](https://docs.nestjs.com/)

### Why Use a Framework?

- In the earlier TypeScript and Node.js material, we built a RESTful API by hand
- Notice what happens without a framework: every person — and every company — solves the same problem in a different way, even when the end result is identical
- There is no agreed-upon structure, so everyone works however they like
- A framework forces everyone onto the same structure

**What you gain:**

- Everyone works the same way — no more arguing about project structure
- New hires onboard quickly, because the structure is already familiar to them
- Cross-cutting concerns (validation, logging, error handling, auth) have one obvious place to live instead of being scattered

### What NestJS Uses Under the Hood

NestJS is not a from-scratch reinvention. Internally it leans on libraries that are already standard in the Node.js world:

| Concern | Library NestJS uses |
| --- | --- |
| HTTP handling | **Express** by default (**Fastify** is a drop-in alternative via `@nestjs/platform-fastify`) |
| Unit & E2E testing | **Jest**, plus **Supertest** for HTTP assertions |
| Database / ORM | integrates with **Prisma**, TypeORM, MikroORM, Sequelize, Mongoose |
| Logging | its own `Logger`, swappable for **Winston**, Pino, etc. |

Because of this, learning NestJS is mostly a matter of learning how it *wires together* libraries you already know.

> **Key Insight:** NestJS is the *structure*, not the *engine*. Almost every capability is a well-known Node library that NestJS exposes through modules and dependency injection.

---

## 🏗️ Project Setup

### Installing the NestJS CLI

Setting up a TypeScript project by hand is tedious — there is a lot to configure. NestJS ships a CLI that does it for you.

```bash
npm install -g @nestjs/cli
```

Once installed, the CLI is available in your terminal:

```bash
nest
```

> Reference: [github.com/nestjs/nest-cli](https://github.com/nestjs/nest-cli)

### Creating a Project

```bash
nest new nama-project
```

The CLI asks a few questions (most importantly which package manager to use), then generates the `nama-project` folder with a working NestJS skeleton inside.

Two other generators are worth knowing early:

```bash
# Generate a single building block
nest generate module user
nest generate controller user
nest generate service user

# Scaffold a complete CRUD resource (module + controller + service + DTOs + tests)
nest generate resource user
```

> **Tip:** Add `--dry-run` to any `nest generate` command to preview the files it would create without touching the disk.

### Folder Structure

| Path | Contents |
| --- | --- |
| `/src` | Application code and unit tests (`*.spec.ts`) |
| `/test` | Integration / end-to-end tests (`*.e2e-spec.ts`) |

The structure is deliberately small, so there isn't much to learn before you're productive.

### Running the Application

Every instruction for building, running in development, running in production, and testing already lives in the generated `README.md`. For the full picture, look at the `scripts` section of `package.json`:

```bash
npm run start          # run once
npm run start:dev      # watch mode — restart on change
npm run start:debug    # watch mode + debugger attached
npm run build          # compile to /dist
npm run start:prod     # run the compiled output
npm run test           # unit tests
npm run test:e2e       # end-to-end tests
npm run test:cov       # coverage report
```

---

## 🏷️ Decorators

### What a Decorator Is

The single biggest difference between NestJS and hand-written Express is the **Decorator**. NestJS uses decorators almost everywhere.

- A decorator attaches **metadata** (extra information) to your code
- Write `@` followed by the decorator name — this is also called an *annotation*
- Decorators can be applied to classes, methods, properties, parameters, and constructor arguments
- The JavaScript decorator proposal is still not finalized by everyone, but it is already widely used in practice — NestJS being one of the biggest adopters

> Reference: [github.com/tc39/proposal-decorators](https://github.com/tc39/proposal-decorators)

> **Note:** Because decorators emit metadata at runtime, a NestJS project needs `"experimentalDecorators": true` and `"emitDecoratorMetadata": true` in `tsconfig.json`, plus `import 'reflect-metadata'` — the CLI sets all of this up for you.

### Where Decorators Appear in NestJS

| Level | Examples |
| --- | --- |
| Class | `@Module()`, `@Controller()`, `@Injectable()`, `@Catch()`, `@Global()` |
| Method | `@Get()`, `@Post()`, `@HttpCode()`, `@UseGuards()`, `@UseFilters()` |
| Parameter | `@Query()`, `@Param()`, `@Body()`, `@Req()`, `@Inject()` |
| Property | `@Inject()`, `@Optional()` |

---

## 📦 Modules

A **Module** is a class annotated with `@Module()`. NestJS splits an application into modules, and this modular structure is the backbone of the whole framework.

- Every NestJS application has one **root module** (the *Application Module*, usually `AppModule`)
- The root module then imports every other module in the app
- Modules are **singletons** — importing the same module from ten places still creates it only once

### The Application Module

The `@Module()` decorator takes four properties:

| Property | Purpose |
| --- | --- |
| `imports` | Other modules whose exported providers this module needs |
| `controllers` | Controllers this module instantiates |
| `providers` | Providers (services, repositories, factories…) available inside this module |
| `exports` | The subset of `providers` that other modules are allowed to use |

### Creating a Module

You can write the class by hand and add it to the root module's `imports`, or let the CLI do it:

```bash
nest generate module <name> [path]
```

If the module lives directly under `src`, the path can be omitted:

```console
study-nestjs-basic on  main [?] via  v24.4.0
❯ nest generate module user
CREATE src/user/user.module.ts (81 bytes)
UPDATE src/app.module.ts (308 bytes)
```

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
import { Module } from '@nestjs/common';

@Module({})
export class UserModule {}
```

The generated module starts empty — the CLI fills in `controllers` and `providers` as you generate them. It also automatically registers `UserModule` in `AppModule`'s `imports`.

---

## 🎮 Controllers

A **Controller** is a class annotated with `@Controller()`. Its job is to receive an incoming HTTP request and return an HTTP response.

- After creating a controller, it must be registered in a module's `controllers` array
- `@Controller()` accepts a **prefix path**, e.g. `@Controller('/api/users')` — every route in that class is then relative to `/api/users`

### Creating a Controller

Write it by hand, or generate it:

```bash
nest generate controller <name> [path]
```

When you use the CLI, the controller is automatically registered in the module that lives at the same path.

```console
study-nestjs-basic on  main [?] via  v24.4.0
❯ nest generate controller user user
CREATE src/user/user/user.controller.spec.ts (478 bytes)
CREATE src/user/user/user.controller.ts (97 bytes)
UPDATE src/user/user.module.ts (171 bytes)
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
import { Controller } from '@nestjs/common';

@Controller('/api/users')
export class UserController {}
```

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { UserController } from './user/user.controller';

@Module({
  controllers: [UserController],
})
export class UserModule {}
```

### Routing & HTTP Method Decorators

Routing is simple: add a method to the controller and annotate it with the HTTP method decorator you want.

| Decorator | HTTP method |
| --- | --- |
| `@Get(path?)` | GET |
| `@Post(path?)` | POST |
| `@Put(path?)` | PUT |
| `@Delete(path?)` | DELETE |
| `@Patch(path?)` | PATCH |
| `@Head(path?)` | HEAD |
| `@Options(path?)` | OPTIONS |
| `@All(path?)` | every HTTP method |

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
import { Controller, Get, Post } from '@nestjs/common';

@Controller('/api/users')
export class UserController {
  @Post()
  post(): string {
    return 'POST';
  }

  @Get('/sample')
  get(): string {
    return 'GET';
  }
}
```

> **Note:** NestJS returns **200** for a successful GET and **201** for a successful POST by default. Use `@HttpCode()` to override it.

### Route Order Matters

Routes are matched **in declaration order**, so a dynamic segment declared early will swallow the static routes that come after it:

```ts
@Get('/:id')          // ⚠️ declared first — matches literally anything
getById(@Param('id') id: string): string {
  return `GET ${id}`;
}

@Get('/sample')       // ❌ unreachable — '/sample' already matched '/:id'
get(): string {
  return 'GET';
}
```

Always declare **specific routes before dynamic ones**:

```ts
@Get('/sample')       // ✅ specific first
get(): string {
  return 'GET';
}

@Get('/:id')          // ✅ catch-all last
getById(@Param('id') id: string): string {
  return `GET ${id}`;
}
```

This becomes a hard failure once you add a pipe: with `@Param('id', ParseIntPipe)`, a request to `/api/users/sample` hits the `/:id` handler and returns **400 Bad Request** because `"sample"` is not a number.

---

## 📥 Handling the Request

### The Express Request Object

When writing routes you usually need data the client sent — query params, headers, the request body, and so on.

Because NestJS runs on Express by default, you can take an `express.Request` parameter and annotate it with `@Req()`.

> **Recommendation:** Even though the raw `express.Request` object is available, prefer the dedicated decorators below. They state exactly what the handler needs, keep the handler decoupled from Express, and make unit testing far easier — you pass plain values instead of building a mock request.

### Request Decorators

| Decorator | Express equivalent |
| --- | --- |
| `@Req()` | `req` (the whole `express.Request`) |
| `@Param(key?)` | `req.params` / `req.params[key]` |
| `@Body(key?)` | `req.body` / `req.body[key]` |
| `@Query(key?)` | `req.query` / `req.query[key]` |
| `@Headers(key?)` | `req.headers` / `req.headers[key]` |
| `@Ip()` | `req.ip` |
| `@HostParam()` | `req.hosts` |
| `@Session()` | `req.session` |

> **Correction:** the request-header decorator is **`@Headers()`** (plural). `@Header(key, value)` is a *response* decorator that sets an outgoing header — they look almost identical but do opposite things.

### Reading Query & Route Params

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
import { Controller, Get, Param, Post, Query } from '@nestjs/common';

@Controller('/api/users')
export class UserController {
  @Get('/hello')
  sayHello(
    @Query('first_name') firstName: string,
    @Query('last_name') lastName: string,
  ): string {
    return `Hello ${firstName} ${lastName}`;
  }

  @Get('/:id')
  getById(@Param('id') id: string): string {
    return `GET ${id}`;
  }

  @Post()
  post(): string {
    return 'POST';
  }
}
```

> **Note:** query and route params always arrive as **strings**. Converting them to a number, boolean, or `Date` is exactly what [Pipes](#-pipes) are for.

---

## 📤 Shaping the Response

### The Express Response Object

- By default, whatever a controller method **returns** becomes the HTTP response body
- You can still use `express.Response` by annotating a parameter with `@Res()`
- ⚠️ The moment you inject `@Res()`, you take over: NestJS stops handling the response, so you **must** send it yourself — the return value is ignored

> **Recommendation:** stick to the return value. Use the response decorators below to adjust status codes and headers, and reach for `@Res()` only when you genuinely need something Express-specific (streaming, `res.render()`, `res.cookie()`).

### Response Decorators

| Decorator | Purpose |
| --- | --- |
| `@HttpCode(code)` | Change the response status code |
| `@Header(key, value)` | Set a response header |
| `@Redirect(location, code)` | Redirect; the target can be overridden by returning an `HttpRedirectResponse` |
| `@Res()` | Inject the raw `express.Response` |
| `@Next()` | Inject `express.NextFunction` |

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
import {
  Controller,
  Get,
  Header,
  HttpCode,
  Param,
  Post,
  Query,
  Redirect,
} from '@nestjs/common';
import type { HttpRedirectResponse } from '@nestjs/common';

@Controller('/api/users')
export class UserController {
  // ❌ NOT RECOMMENDED — takes over the response, harder to test
  // @Get('/sample-response')
  // sampleResponse(@Res() response: Response) {
  //   response.status(200).send('Sample response');
  // }

  // ✅ RECOMMENDED — return a value, describe the rest with decorators
  @Get('/sample-response')
  @Header('Content-Type', 'application/json')
  @HttpCode(200)
  sampleResponse(): Record<string, string> {
    return { message: 'Sample response' };
  }

  @Get('/redirect')
  @Redirect()
  redirect(): HttpRedirectResponse {
    return {
      url: '/api/users/sample-response',
      statusCode: 301,
    };
  }
}
```

### Asynchronous Handlers

Just like Express handlers, NestJS controller methods can be `async`. Return a `Promise<T>` and NestJS resolves it before sending the response.

```ts
@Get('/hello')
async sayHello(
  @Query('first_name') firstName: string,
  @Query('last_name') lastName: string,
): Promise<string> {
  await new Promise((resolve) => setTimeout(resolve, 1000));
  return `Hello ${firstName} ${lastName}`;
}
```

> NestJS also accepts RxJS `Observable` return values — it subscribes and sends the emitted value automatically.

---

## 🍪 Cookies

Express has no built-in cookie support, so — exactly as in the Express material — you add the `cookie-parser` middleware. In NestJS you register it on the app inside `main.ts`.

```bash
npm install cookie-parser
npm install --save-dev @types/cookie-parser
```

**`study-nestjs-basic/src/main.ts`**

```ts
const app = await NestFactory.create<NestExpressApplication>(AppModule);
app.use(cookieParser('secret123'));
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
import { Controller, Get, Query, Req, Res } from '@nestjs/common';
import type { Request, Response } from 'express';

@Controller('/api/users')
export class UserController {
  @Get('/set-cookie')
  setCookie(@Query('name') name: string, @Res() response: Response) {
    response.cookie('name', name);
    response.status(200).send('Success set cookie');
  }

  @Get('/get-cookie')
  getCookie(@Req() request: Request): string {
    return request.cookies?.['name'] ?? 'Cookie not found';
  }
}
```

> **Correction:** import `Request` from `express`, not the global DOM `Request`. In the original notes only `Response` was imported from Express, so `Request` resolved to the browser type and needed an awkward cast (`request as Request & { cookies?: ... }`). Importing the Express type removes the cast entirely.

> **Note:** the `'secret123'` argument enables **signed** cookies, but they only work if you opt in per cookie — `response.cookie('name', name, { signed: true })`, read back via `request.signedCookies`.

---

## 🖼️ Views & Template Engines

NestJS has no template engine of its own, but since it runs on Express you can plug in any Express-compatible engine. Here we reuse **Mustache**, the same engine from the Express material.

```bash
npm install mustache-express
npm install --save-dev @types/mustache-express
```

**`study-nestjs-basic/src/main.ts`**

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express';
import cookieParser from 'cookie-parser';
import mustache from 'mustache-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.use(cookieParser('secret123'));

  app.set('views', __dirname + '/../views');
  app.set('view engine', 'html');
  app.engine('html', mustache());

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

> **Key Insight:** `NestFactory.create<NestExpressApplication>()` is what unlocks `app.set()` and `app.engine()`. Without that generic argument you only get the platform-agnostic `INestApplication`, which has no Express-specific methods.

> **Gotcha:** `__dirname` does **not** exist in ES modules. This project sets `"type": "module"` in `package.json`, so once you hit the ESM build you have to recreate it — see the [Logging](#-logging-with-winston) section, where `main.ts` adds `const __dirname = dirname(fileURLToPath(import.meta.url))`.

**`study-nestjs-basic/views/index.html`**

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ title }}</title>
</head>

<body>
  <h1>Hello {{ name }}</h1>
</body>

</html>
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/view-hello')
viewHello(@Query('name') name: string, @Res() response: Response) {
  response.render('index', {
    title: 'Template Engine',
    name: name,
  });
}
```

Rendering is one of the legitimate reasons to reach for `@Res()` — `render()` lives on the Express response object.

---

## 🧪 Testing

### Unit Testing

When you generate a controller or service with the CLI, a unit test template comes with it. NestJS uses **Jest**, so everything from the Node.js unit-testing material applies directly.

The core building block is `Test.createTestingModule()` — it builds a miniature NestJS module containing only what the test needs.

**`study-nestjs-basic/src/user/user/user.service.spec.ts`**

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UserService],
    }).compile();

    service = module.get<UserService>(UserService);
  });

  it('should be able to say hello', () => {
    const response = service.sayHello('Dzaru');
    expect(response).toBe('Hello Dzaru');
  });
});
```

### Mocking Request & Response

Unit testing gets awkward the moment a controller takes an `express.Request` or `express.Response` parameter — you now need a **mock object**, exactly as covered in the Node.js unit-testing material.

```bash
npm install --save-dev node-mocks-http
```

> Reference: [npmjs.com/package/node-mocks-http](https://www.npmjs.com/package/node-mocks-http)

**`study-nestjs-basic/src/user/user/user.controller.spec.ts`**

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserController } from './user.controller';
import * as httpMocks from 'node-mocks-http';
import { UserService } from './user.service';

describe('UserController', () => {
  let controller: UserController;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [UserService],
    }).compile();

    controller = module.get<UserController>(UserController);
  });

  it('should can say hello', () => {
    const response = controller.sayHello('Dzaru', 'Rizky');
    expect(response).toBe('Hello Dzaru Rizky');
  });

  it('should can view template', () => {
    const response = httpMocks.createResponse();
    controller.viewHello('Dzaru', response);

    expect(response._getRenderView()).toBe('index');
    expect(response._getRenderData()).toEqual({
      name: 'Dzaru',
      title: 'Template Engine',
    });
  });
});
```

Notice that any provider a controller depends on has to be listed in `providers` too, otherwise NestJS cannot resolve the constructor. To swap in a fake instead, use `overrideProvider`:

```ts
const module = await Test.createTestingModule({
  controllers: [UserController],
  providers: [UserService],
})
  .overrideProvider(UserService)
  .useValue({ sayHello: () => 'Hello Mock' })
  .compile();
```

> **Key Insight:** this is the payoff for preferring `@Query()`/`@Body()` over `@Req()`. `sayHello('Dzaru', 'Rizky')` needs no mocking at all, while `viewHello` needs `node-mocks-http` purely because it accepts `@Res()`.

### Integration (E2E) Testing

`nest new` also scaffolds integration tests in the `test` folder. They use **Jest + Supertest**, both already familiar from the Node.js and TypeScript material.

- By convention, integration test files end with `.e2e-spec.ts` (**e2e** = *end to end*)
- Configuration lives in `test/jest-e2e.json`

**`study-nestjs-basic/test/app.e2e-spec.ts`**

```ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { App } from 'supertest/types';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication<App>;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/ (GET)', () => {
    return request(app.getHttpServer())
      .get('/')
      .expect(200)
      .expect('Hello World!');
  });

  it('should can say hello', async () => {
    await request(app.getHttpServer())
      .get('/api/users/hello')
      .query({ first_name: 'Dzaru', last_name: 'Rizky' })
      .expect(200)
      .expect('Hello Dzaru Rizky');
  });

  afterEach(async () => {
    await app.close();
  });
});
```

The difference from a unit test: here the whole `AppModule` boots, so real middleware, guards, pipes, interceptors and filters all run — which is exactly the point.

> **Note:** `await app.close()` in `afterEach` is not optional. Skip it and the HTTP server (and any DB connection) stays open, and Jest hangs with *"a worker process has failed to exit gracefully"*.

---

## 🧩 Providers & Dependency Injection

**Provider** is one of the most fundamental concepts in NestJS. Most classes in an application qualify: services, repositories, factories, helpers, and so on.

The whole reason the concept exists is so an object can be **injected as a dependency** into another object — a controller, or another provider.

### Creating a Provider

- Create a class and annotate it with `@Injectable()`
- Register it in a module's `providers` array so NestJS can instantiate it

```bash
nest generate provider <name> [path]

# For services specifically:
nest generate service <name> [path]
```

```console
study-nestjs-basic on  main [?] via  v24.4.0
❯ nest generate service user user
CREATE src/user/user/user.service.spec.ts (446 bytes)
CREATE src/user/user/user.service.ts (88 bytes)
UPDATE src/user/user.module.ts (250 bytes)
```

**`study-nestjs-basic/src/user/user/user.service.ts`**

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  sayHello(name: string): string {
    return `Hello ${name}`;
  }
}
```

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { UserController } from './user/user.controller';
import { UserService } from './user/user.service';

@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}
```

### Constructor Injection

**Dependency Injection** is the idea that an object needs another object — `UserController` needs `UserService`, for example.

NestJS wires this automatically. Declare what you need in the constructor and NestJS injects it:

```ts
@Controller('/api/users')
export class UserController {
  constructor(private service: UserService) {}

  @Get('/hello')
  sayHello(
    @Query('first_name') firstName: string,
    @Query('last_name') lastName: string,
  ): string {
    return this.service.sayHello(`${firstName} ${lastName}`);
  }
}
```

- The `private` modifier is TypeScript shorthand — it declares and assigns `this.service` in one line
- Modules, controllers, and providers are all created as **singletons** by default

> **Key Insight:** NestJS resolves constructor dependencies by *type*, which only works because `emitDecoratorMetadata` writes the parameter types into the compiled output. That's also why interfaces can't be injected directly — they don't exist at runtime, so you need a string/symbol token instead (see [Alias Provider](#alias-provider)).

### Property-Based Injection

Constructor parameters are the default, but NestJS also supports injecting through a property. Annotate it with `@Inject()`:

```ts
@Injectable()
export class UserService {
  @Inject('EmailService')
  private emailService: MailService;
}
```

> Prefer constructor injection. Property injection makes dependencies invisible to whoever constructs the class manually — which mainly hurts in tests. Reach for it only when a subclass would otherwise have to forward every constructor argument.

### Optional Dependencies

By default a dependency is mandatory: if NestJS cannot find it, startup fails with an error. When a dependency really is optional, annotate it with `@Optional()`:

```ts
constructor(@Optional() private mailService?: MailService) {}
```

NestJS then injects `undefined` instead of throwing.

---

## 🏭 Custom Providers

Sometimes a provider is too complex to declare by class name alone. Or it *can't* be — the class belongs to a third-party library and you cannot add `@Injectable()` to it (`RedisClient`, `MongoClient`, `PrismaClient`, …).

NestJS supports several ways of declaring a provider. All of them are objects with a `provide` key (the **token**) plus one `useX` key (the **recipe**).

| Form | Key | Use it when |
| --- | --- | --- |
| Standard | *(class name only)* | The plain case — NestJS instantiates the class for you |
| Class | `useClass` | The token stays the same but the implementation varies |
| Value | `useValue` | You already have the object (or want a mock in tests) |
| Factory | `useFactory` | Creating the object needs logic and/or other dependencies |
| Alias | `useExisting` | A second name for a provider that already exists |

The example below builds all five around a `Connection` abstraction.

**`study-nestjs-basic/src/user/connection/connection.ts`**

```ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class Connection {
  getName(): string | undefined {
    return undefined;
  }
}

@Injectable()
export class MySQLConnection implements Connection {
  getName(): string | undefined {
    return 'MySQL';
  }
}

@Injectable()
export class MongoDBConnection implements Connection {
  getName(): string | undefined {
    return 'MongoDB';
  }
}

export function createConnection(configService: ConfigService): Connection {
  if (configService.get('DB_TYPE') === 'mysql') {
    return new MySQLConnection();
  }

  if (configService.get('DB_TYPE') === 'mongodb') {
    return new MongoDBConnection();
  }

  throw new Error('Invalid DB_TYPE');
}
```

**`study-nestjs-basic/src/user/mail/mail.service.ts`**

```ts
export class MailService {
  send(): void {
    console.info('Send email');
  }
}

export const mailService = new MailService();
```

> **Correction:** in the original notes this file contained a copy of `connection.ts`. It has to export a `MailService` class *and* a ready-made `mailService` instance, because the module registers it with `useValue: mailService`.

**`study-nestjs-basic/src/user/user-repository/user-repository.ts`**

```ts
import { Connection } from './../connection/connection';

export class UserRepository {
  connection: Connection;

  save() {
    console.info(`save user with connection ${this.connection.getName()}`);
  }
}

export function createUserRepository(connection: Connection): UserRepository {
  const repository = new UserRepository();
  repository.connection = connection;
  return repository;
}
```

### Standard Provider

The plain form you've already used — just name the class in `providers`. NestJS reads its constructor and builds it.

```ts
providers: [UserService]
```

It is shorthand for `{ provide: UserService, useClass: UserService }`.

### Class Provider

Pick which class satisfies a token. Perfect when one abstraction has several implementations — a `Connection` interface with `MySQLConnection` and `MongoDBConnection` behind it.

```ts
{
  provide: Connection,
  useClass: MySQLConnection,
}
```

### Value Provider

Build a dependency from an object that already exists. Also the easiest way to inject a mock in tests.

```ts
{
  provide: MailService,
  useValue: mailService,
}
```

### Factory Provider

Supply a function — a **factory method** — that NestJS calls to build the object. If the factory itself needs dependencies, list them in `inject`; they are passed as arguments in the same order.

```ts
{
  provide: Connection,
  inject: [ConfigService],
  useFactory: createConnection,
}
```

Factories may be `async` — NestJS awaits the returned promise before the app finishes booting.

### Alias Provider

Give the same underlying object a second name. Useful when different parts of the app expect different provider names for one dependency.

```ts
{
  provide: 'EmailService',
  useExisting: MailService,
}
```

Because `'EmailService'` is a string token rather than a class, injecting it requires `@Inject('EmailService')`:

```ts
constructor(
  @Inject('EmailService')
  private emailService: MailService,
) {}
```

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { UserController } from './user/user.controller';
import { UserService } from './user/user.service';
import { Connection, createConnection } from './connection/connection';
import { mailService, MailService } from './mail/mail.service';
import {
  createUserRepository,
  UserRepository,
} from './user-repository/user-repository';

@Module({
  controllers: [UserController],
  providers: [
    UserService,
    {
      provide: Connection,
      inject: [ConfigService],
      useFactory: createConnection,
    },
    {
      provide: MailService,
      useValue: mailService,
    },
    {
      provide: UserRepository,
      inject: [Connection],
      useFactory: createUserRepository,
    },
    {
      provide: 'EmailService',
      useExisting: MailService,
    },
  ],
})
export class UserModule {}
```

---

## 🔍 Module Reference

NestJS provides a class called **`ModuleRef`** for fetching providers from the container manually.

This helps when you don't want automatic dependency injection — for example, when a dependency should only be resolved at the moment it's actually needed.

**`study-nestjs-basic/src/user/member/member.service.ts`**

```ts
import { Injectable } from '@nestjs/common';
import { ModuleRef } from '@nestjs/core';
import { Connection } from '../connection/connection';
import { MailService } from '../mail/mail.service';

@Injectable()
export class MemberService {
  constructor(private moduleRef: ModuleRef) {}

  getConnectionName(): string | undefined {
    const connection = this.moduleRef.get(Connection);
    return connection.getName();
  }

  sendEmail() {
    const mailService = this.moduleRef.get(MailService);
    mailService.send();
  }
}
```

- `moduleRef.get(Token)` looks inside the current module
- `moduleRef.get(Token, { strict: false })` searches the whole application
- `moduleRef.resolve(Token)` is the counterpart for scoped (non-singleton) providers, and returns a `Promise`

> Use `ModuleRef` sparingly. It hides dependencies from the constructor, which is exactly the coupling that dependency injection is meant to remove. Reach for it for genuinely dynamic lookups, not as a shortcut around the constructor.

---

## ⚙️ Configuration

Every application needs dynamic configuration — a database username and password, for instance.

- A good practice is to use **environment variables**, readable in Node.js via `process.env`
- Beyond raw environment variables, the Node.js convention is a **`.env` file** (Prisma works this way too)
- NestJS can read `.env` files for you

```bash
npm install @nestjs/config
```

Register `ConfigModule` in the application module, then read values through `ConfigService`.

**`study-nestjs-basic/src/app.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

`isGlobal: true` makes `ConfigService` injectable everywhere without importing `ConfigModule` into each module. Other options worth knowing:

| Option | Purpose |
| --- | --- |
| `envFilePath` | Point at a specific file, or several (`['.env.local', '.env']`) |
| `ignoreEnvFile` | Read only real environment variables — the right choice in production |
| `validationSchema` | Validate the environment at boot (Joi) so a missing variable fails fast |
| `load` | Register typed/namespaced config factories |

### Reading Config from the Application

Besides serving requests, the NestJS application object can also **fetch providers**. That's how you make something like the HTTP port configurable:

**`study-nestjs-basic/src/main.ts`**

```ts
const configService = app.get(ConfigService);
const port = configService.get<number>('PORT') ?? 3000;
const host = configService.get<string>('HOST') ?? 'localhost';

await app.listen(port, host);
```

> **Note:** values read from a `.env` file are always **strings**. `configService.get<number>('PORT')` only *asserts* the type — it does not convert it. Use `Number(...)`, or a `validationSchema` that coerces, if you need a real number.

---

## 🔗 Shared Modules

Importing one module from another is straightforward, and since a module is a **singleton**, you never have to worry about a module imported by ten other modules being created ten times — it is created exactly once.

Use the `imports` array to bring a module in.

### Sharing Providers

- By default, providers are **not** shared outside their own module
- A provider is only usable inside the module that declares it
- To share one, list it in the module's `exports` array
- Only exported providers are visible to other modules

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
@Module({
  controllers: [UserController],
  providers: [
    UserService,
    // ...other providers
  ],
  exports: [UserService],
})
export class UserModule {}
```

**`study-nestjs-basic/src/app.controller.ts`**

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';
import { UserService } from './user/user/user.service';

@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService,
    private userService: UserService,
  ) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

`AppController` can inject `UserService` only because `AppModule` imports `UserModule` **and** `UserModule` exports `UserService`. Drop either half and startup fails with `Nest can't resolve dependencies of the AppController`.

> **Key Insight:** `exports` is a module's public API. Providers you don't export are private implementation details — which is a feature, not a limitation.

---

## 🗄️ Database with Prisma

NestJS provides no database or ORM (Object Relational Mapping) layer of its own. It integrates with all the popular ones instead:

> Reference: [MikroORM](https://docs.nestjs.com/recipes/mikroorm) · [TypeORM](https://docs.nestjs.com/recipes/sql-typeorm) · [MongoDB](https://docs.nestjs.com/recipes/mongodb) · [Sequelize](https://docs.nestjs.com/recipes/sql-sequelize) · [Prisma](https://docs.nestjs.com/recipes/prisma)

We already covered Prisma in the Node.js material, so we'll wire Prisma into the NestJS app we've been building.

```bash
npm install --save-dev prisma
npx prisma init
```

**`study-nestjs-basic/src/prisma/contract.prisma`**

```prisma
model User {
  id         Int @id @default(autoincrement())
  first_name VarChar(100)
  last_name  VarChar(100)
  email      VarChar(100)

  @@map("users")
}
```

The idiomatic NestJS integration is a small module wrapping the client as an injectable provider:

**`study-nestjs-basic/src/prisma/prisma.service.ts`**

```ts
import { Injectable } from '@nestjs/common';
import { db } from './db.js';

@Injectable()
export class PrismaService {
  db = db;

  constructor() {
    console.info(`Create prisma service at ${new Date().toISOString()}`);
  }
}
```

**`study-nestjs-basic/src/prisma/prisma.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service.js';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

`PrismaModule` exports `PrismaService`, so any module that imports it can inject the client:

**`study-nestjs-basic/src/user/user.module.ts`**

```ts
@Module({
  imports: [PrismaModule],
  controllers: [UserController],
  providers: [
    UserService,
    {
      provide: Connection,
      inject: [ConfigService],
      useFactory: createConnection,
    },
    {
      provide: MailService,
      useValue: mailService,
    },
    {
      provide: 'EmailService',
      useExisting: MailService,
    },
    UserRepository,
    MemberService,
  ],
  exports: [UserService],
})
export class UserModule {}
```

**`study-nestjs-basic/src/user/user-repository/user-repository.ts`**

```ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from './../../prisma/prisma.service.js';
import type { FieldOutputTypes } from './../../prisma/contract.js';

export type User = FieldOutputTypes['public']['User'];

@Injectable()
export class UserRepository {
  constructor(private prismaService: PrismaService) {
    console.info('Create user repository at ' + new Date().toISOString());
  }

  async save(
    firstName: string,
    lastName: string,
    email: string,
  ): Promise<User> {
    const plan = this.prismaService.db.sql.public.users
      .insert([{ first_name: firstName, last_name: lastName, email }])
      .returning('id', 'first_name', 'last_name', 'email')
      .build();
    const [row] = await this.prismaService.db.runtime().query(plan);
    return row;
  }
}
```

> **Note:** `UserRepository` switches from a **factory provider** to a **standard provider** here. Once it carries `@Injectable()` and declares its dependency in the constructor, NestJS can build it directly — `createUserRepository` is no longer needed.

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/create')
async create(
  @Query('first_name') firstName: string,
  @Query('last_name') lastName: string,
  @Query('email') email: string,
): Promise<User> {
  const user = await this.userRepository.save(firstName, lastName, email);
  return user;
}
```

> ⚠️ `/create` is a **GET** route that writes to the database. That's fine for a study project, but in real code a write belongs behind `@Post()` with a `@Body()` payload — GET requests are supposed to be safe and are freely cached and retried.

---

## 📝 Logging with Winston

NestJS ships its own `Logger`, used during startup and whenever an error occurs. By default it logs plain text to the console.

Sometimes you want something better — **Winston**, for instance. NestJS lets you replace the logger by supplying your own `LoggerService`, and the `nest-winston` package does exactly that.

```bash
npm install nest-winston winston
```

> Reference: [github.com/gremo/nest-winston](https://github.com/gremo/nest-winston/blob/main/src/winston.classes.ts)

**`study-nestjs-basic/src/app.module.ts`**

```ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller.js';
import { AppService } from './app.service.js';
import { UserModule } from './user/user.module.js';
import { PrismaModule } from './prisma/prisma.module.js';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
    PrismaModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**`study-nestjs-basic/src/main.ts`**

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module.js';
import { NestExpressApplication } from '@nestjs/platform-express';
import cookieParser from 'cookie-parser';
import mustache from 'mustache-express';
import { ConfigService } from '@nestjs/config';
import { dirname } from 'path';
import { fileURLToPath } from 'url';
import { WINSTON_MODULE_NEST_PROVIDER, WinstonLogger } from 'nest-winston';

const __dirname = dirname(fileURLToPath(import.meta.url));

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.use(cookieParser('secret123'));

  const loggerService = app.get<WinstonLogger>(WINSTON_MODULE_NEST_PROVIDER);
  app.useLogger(loggerService);

  app.set('views', __dirname + '/../views');
  app.set('view engine', 'html');
  app.engine('html', mustache());

  const configService = app.get(ConfigService);
  const port = configService.get<number>('PORT') ?? 3000;
  const host = configService.get<string>('HOST') ?? 'localhost';

  await app.listen(port, host);
}
bootstrap().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

`nest-winston` exposes **two different tokens**, and picking the wrong one is a common mistake:

| Token | What you get | Use it for |
| --- | --- | --- |
| `WINSTON_MODULE_NEST_PROVIDER` | A `WinstonLogger` adapter implementing NestJS's `LoggerService` | `app.useLogger(...)` — replacing the framework logger |
| `WINSTON_MODULE_PROVIDER` | The raw `winston.Logger` | Injecting into your own services for application logging |

**`study-nestjs-basic/src/user/user-repository/user-repository.ts`**

```ts
import { Inject, Injectable } from '@nestjs/common';
import { PrismaService } from './../../prisma/prisma.service.js';
import type { FieldOutputTypes } from './../../prisma/contract.js';
import { Logger } from 'winston';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';

export type User = FieldOutputTypes['public']['User'];

@Injectable()
export class UserRepository {
  constructor(
    private prismaService: PrismaService,
    @Inject(WINSTON_MODULE_PROVIDER) private logger: Logger,
  ) {
    this.logger.info('Create user repository at ' + new Date().toISOString());
  }

  async save(
    firstName: string,
    lastName: string,
    email: string,
  ): Promise<User> {
    this.logger.info(
      `Save user with first name: ${firstName}, last name: ${lastName}, email: ${email}`,
    );
    const plan = this.prismaService.db.sql.public.users
      .insert([{ first_name: firstName, last_name: lastName, email }])
      .returning('id', 'first_name', 'last_name', 'email')
      .build();
    this.logger.info('SQL query: ' + JSON.stringify(plan));
    const [row] = await this.prismaService.db.runtime().query(plan);
    return row;
  }
}
```

Since `WINSTON_MODULE_PROVIDER` is a string token, injecting it requires `@Inject()`.

---

## 🌍 Global Modules

If a module is needed by many other modules, you can promote it to a **global module** — it is then imported into every module automatically.

- Annotate the module class with `@Global()`
- `nest-winston` does exactly this, which is why `WinstonModule` never needs importing anywhere else

> Reference: [nest-winston's global module](https://github.com/gremo/nest-winston/blob/main/src/winston.module.ts#L5)

**`study-nestjs-basic/src/prisma/prisma.module.ts`**

```ts
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service.js';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

**`study-nestjs-basic/src/app.module.ts`**

```ts
@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Notice `PrismaModule` disappeared from `UserModule`'s `imports` — `@Global()` handles it.

> ⚠️ A global module still has to be registered **once** (typically in the root module) for its providers to exist at all. And the NestJS docs are explicit that making everything global is poor design: `imports` is what documents a module's dependencies, and `@Global()` erases that. Reserve it for genuinely cross-cutting infrastructure — a database connection, a logger, config.

---

## 🔄 Dynamic Modules

You may have noticed that `ConfigModule` and `WinstonModule` are configured by calling a method (`.forRoot(...)`) rather than being listed directly. That's a **dynamic module**.

A dynamic module is built at runtime, so its providers and controllers can depend on parameters the caller supplies.

- Create a static method that returns a `DynamicModule` object
- Everything available on a regular `@Module()` is available here too

> Reference: [`DynamicModule` interface](https://github.com/nestjs/nest/blob/master/packages/common/interfaces/modules/dynamic-module.interface.ts) · [`ModuleMetadata` interface](https://github.com/nestjs/nest/blob/master/packages/common/interfaces/modules/module-metadata.interface.ts)

| Convention | Meaning |
| --- | --- |
| `forRoot()` | Configure once, at the root of the application |
| `forFeature()` | Per-feature configuration, called by individual modules |
| `forRootAsync()` | Same as `forRoot()`, but the options themselves are resolved asynchronously (often from `ConfigService`) |

The concrete example lives in the next section, where validation is wired up as a dynamic module.

---

## ✅ Validation

NestJS has no validation feature of its own. But because everything is a module, adding validation is easy — build a module and register it.

Two libraries dominate in the NestJS community:

> Reference: [class-validator](https://github.com/typestack/class-validator) · [Zod](https://github.com/colinhacks/zod)

We'll use **Zod**, since it was already covered in detail in the TypeScript material.

```bash
npm install zod
```

**`study-nestjs-basic/src/validation/validation.module.ts`**

```ts
import { DynamicModule, Module } from '@nestjs/common';
import { ValidationService } from './validation/validation.service.js';

@Module({})
export class ValidationModule {
  static forRoot(isGlobal: boolean): DynamicModule {
    return {
      module: ValidationModule,
      providers: [ValidationService],
      exports: [ValidationService],
      global: isGlobal,
    };
  }
}
```

This is the dynamic module from the previous section made concrete: the caller decides at runtime whether the module is global.

**`study-nestjs-basic/src/validation/validation/validation.service.ts`**

```ts
import { Injectable } from '@nestjs/common';
import { ZodType } from 'zod';

@Injectable()
export class ValidationService {
  validate<T>(schema: ZodType<T>, data: T): T {
    return schema.parse(data);
  }
}
```

**`study-nestjs-basic/src/user/user/user.service.ts`**

```ts
import z from 'zod';
import { Injectable } from '@nestjs/common';
import { ValidationService } from '../../validation/validation/validation.service.js';

@Injectable()
export class UserService {
  constructor(private validationService: ValidationService) {}

  sayHello(name: string): string {
    const schema = z.string().min(3).max(100);
    const data = this.validationService.validate(schema, name);
    return `Hello ${data}`;
  }
}
```

**`study-nestjs-basic/src/app.module.ts`**

```ts
imports: [
  // ...
  ValidationModule.forRoot(true),
],
```

> **Note:** `schema.parse()` **throws** a `ZodError` when validation fails. That throw is what the [Exception Filter](#-exception-filters) below turns into a clean 400 response. If you'd rather branch on the result instead of throwing, use `schema.safeParse()`, which returns `{ success, data | error }`.

---

## 🔀 Middleware

Middleware in NestJS works the same way as in Express.

- Middleware supports dependency injection, so it can take providers in its constructor
- Create a class implementing `NestMiddleware`, or generate one:

```bash
nest generate middleware <name> [path]
```

### Registering Middleware

- `@Module()` has **no** property for middleware
- Instead, the module class must implement the `NestModule` interface
- Inside `configure(consumer: MiddlewareConsumer)`, declare which middleware runs on which routes
- A route can be specified either as a controller class or as a path
- Paths support `*` as a wildcard, matching any combination of characters

**`study-nestjs-basic/src/log/log.middleware.ts`**

```ts
import { Inject, Injectable, NestMiddleware } from '@nestjs/common';
import type { Request, Response } from 'express';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import type { Logger } from 'winston';

@Injectable()
export class LogMiddleware implements NestMiddleware<Request, Response> {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private readonly logger: Logger,
  ) {}

  use(req: Request, res: Response, next: () => void) {
    this.logger.info(`Receive request for url: ${req.url}`, 'LogMiddleware');
    next();
  }
}
```

**`study-nestjs-basic/src/app.module.ts`**

```ts
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller.js';
import { AppService } from './app.service.js';
import { UserModule } from './user/user.module.js';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { ValidationModule } from './validation/validation.module.js';
import { LogMiddleware } from './log/log.middleware.js';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
    ValidationModule.forRoot(true),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LogMiddleware).forRoutes({
      path: '/api/*',
      method: RequestMethod.ALL,
    });
  }
}
```

> ⚠️ **NestJS 11 / Express 5 change:** bare `*` wildcards are no longer valid — Express 5 requires **named** wildcards. Write `path: '/api/*path'` (or `'/api/{*path}'` to also match `/api` itself). The old `'/api/*'` form silently stops matching what you expect.

> **Key Insight:** middleware is the only piece of the pipeline that runs **before** NestJS knows which route handler was chosen. That's precisely why it can't do authorization — see [Guards](#-guards).

---

## 🚨 Exception Filters

NestJS has a feature called the **Exception Filter**, responsible for handling errors the application didn't catch.

When an unhandled error escapes, the exception filter catches it and automatically sends a friendlier response to the client.

By default NestJS installs a **global exception filter** that converts uncaught errors into a JSON response.

### Creating an Exception Filter

Sometimes the default format isn't what you want. Build your own by implementing `ExceptionFilter` and declaring which error type it handles with `@Catch(ErrorType)`.

**`study-nestjs-basic/src/validation/validation.filter.ts`**

```ts
import { ArgumentsHost, Catch, ExceptionFilter } from '@nestjs/common';
import { ZodError } from 'zod';
import { Response } from 'express';

@Catch(ZodError)
export class ValidationFilter implements ExceptionFilter<ZodError> {
  catch(exception: ZodError, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const response = http.getResponse<Response>();
    response.status(400).json({
      statusCode: 400,
      message: exception.issues,
    });
  }
}
```

### Using a Filter

Apply it with `@UseFilters()`, either on a single method or on the whole controller class. Placed on the class, every route method in it uses that filter.

```ts
@Get('/hello')
@UseFilters(ValidationFilter)
sayHello(@Query('name') name: string): string {
  return this.service.sayHello(name);
}
```

Now a name shorter than 3 characters produces a clean 400 with Zod's issue list instead of a 500.

### HttpException

NestJS provides an error class called **`HttpException`**. Throw it and the global exception handler turns it into an HTTP response matching its contents.

`HttpException` takes a `response` (the body) and a `status` (the HTTP status code):

```ts
@Get('/create')
async create(
  @Query('first_name') firstName: string,
  @Query('last_name') lastName: string,
  @Query('email') email: string,
): Promise<User> {
  if (!firstName) {
    throw new HttpException(
      {
        statusCode: 400,
        message: 'First name is required',
      },
      400,
    );
  }
  const user = await this.userRepository.save(firstName, lastName, email);
  return user;
}
```

NestJS also ships ready-made subclasses that are shorter to write and self-documenting:

| Class | Status |
| --- | --- |
| `BadRequestException` | 400 |
| `UnauthorizedException` | 401 |
| `ForbiddenException` | 403 |
| `NotFoundException` | 404 |
| `ConflictException` | 409 |
| `UnprocessableEntityException` | 422 |
| `InternalServerErrorException` | 500 |

```ts
throw new BadRequestException('First name is required');
```

### Global Exception Filters

To apply a filter across every controller and route, register it on the NestJS application:

**`study-nestjs-basic/src/main.ts`**

```ts
app.useGlobalFilters(new ValidationFilter());
```

Two more things worth knowing:

- `@Catch()` with **no arguments** catches everything — useful for a last-resort fallback filter
- A filter registered with `new` this way cannot use dependency injection. If your filter needs providers, register it with the `APP_FILTER` token instead — see [Global Providers](#-global-providers)

---

## 🚰 Pipes

When a controller method takes a parameter, you usually expect the incoming data to be in a particular shape — a `number`, `string`, `boolean`, `Date`, and so on. But everything arriving over HTTP is a string.

A **Pipe** transforms (and/or validates) a value before it reaches the controller method.

Attach a pipe inside `@Query()`, `@Body()`, or `@Param()`.

### Built-in Pipes

NestJS ships plenty of pipes, so you rarely need to write your own:

| Pipe | Purpose |
| --- | --- |
| `ParseIntPipe` | String → `number` (integer) |
| `ParseFloatPipe` | String → `number` (float) |
| `ParseBoolPipe` | String → `boolean` |
| `ParseArrayPipe` | String → array |
| `ParseUUIDPipe` | Validate a UUID |
| `ParseEnumPipe` | Validate against an enum |
| `ParseDatePipe` | String → `Date` |
| `DefaultValuePipe` | Substitute a default when the value is `undefined` |
| `ParseFilePipe` | Validate uploaded files |

```ts
@Get('/:id')
getById(@Param('id', ParseIntPipe) id: number): string {
  console.info(id * 10);
  return `GET ${id}`;
}
```

With `ParseIntPipe` in place, `id` is a genuine `number` — `id * 10` produces `50` for `/5`, not the string `"5555555555"`.

### Creating a Pipe

When you need something the built-ins don't cover, implement `PipeTransform`:

```bash
nest generate pipe <name> [path]
```

**`study-nestjs-basic/src/validation/validation.pipe.ts`**

```ts
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
import { ZodType } from 'zod';

@Injectable()
export class ValidationPipe implements PipeTransform {
  constructor(private zodType: ZodType) {}

  transform(value: unknown, metadata: ArgumentMetadata) {
    if (metadata.type === 'body') {
      return this.zodType.parse(value);
    }
    return value;
  }
}
```

`metadata.type` tells you where the value came from — `'body'`, `'query'`, `'param'`, or `'custom'`. The guard here matters: applied via `@UsePipes()`, the pipe runs against *every* parameter, and validating a query string against a body schema would fail.

> ⚠️ **Naming collision:** `@nestjs/common` already exports a `ValidationPipe`. A custom class with the same name compiles fine but makes imports genuinely confusing. `ZodValidationPipe` is a safer name.

**`study-nestjs-basic/src/model/login.model.ts`**

```ts
import { z } from 'zod';

export class LoginUserRequest {
  username: string;
  password: string;
}

export const LoginUserRequestValidation = z.object({
  username: z.string().min(10).max(100),
  password: z.string().min(8).max(100),
});
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@UseFilters(ValidationFilter)
@UsePipes(new ValidationPipe(LoginUserRequestValidation))
@Post('/login')
login(
  @Query('name') name: string,
  @Body() // @Body(new ValidationPipe(LoginUserRequestValidation))
  request: LoginUserRequest,
): string {
  return `Hello ${request.username}`;
}
```

### Global Pipes

| Scope | How |
| --- | --- |
| One parameter | `@Body(new ValidationPipe(schema))` |
| One method | `@UsePipes()` on the method |
| One controller | `@UsePipes()` on the class |
| Whole application | `app.useGlobalPipes(...)` in `main.ts` |

> ⚠️ Be careful with global pipes: they run against every parameter in the application, and a type mismatch in one place will start throwing errors in another. That is exactly why the `ValidationPipe` above checks `metadata.type === 'body'` before doing anything.

---

## 🎁 Interceptors

An **Interceptor** is similar to middleware, with one crucial difference: an interceptor can also modify the **response**.

- Middleware only processes the request and hands off to the next middleware or the controller
- An interceptor wraps the handler — it runs code before *and* after, and can transform the value the controller returned before it reaches the client

Beyond response transformation, interceptors are the natural home for cross-cutting concerns: timing/logging, caching, response envelopes, and error mapping.

### Creating an Interceptor

Implement the `NestInterceptor` interface:

```bash
nest generate interceptor <name> [path]
```

Apply it with `@UseInterceptors()` on a method or on the controller class.

### RxJS

NestJS uses **RxJS** for interceptors. If you already know RxJS this is a bonus; if not, don't worry — the usage is simple and feels a lot like working with arrays.

> Reference: [rxjs.dev](https://rxjs.dev/)

**`study-nestjs-basic/src/time/time.interceptor.ts`**

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { map, Observable } from 'rxjs';

@Injectable()
export class TimeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((value: Record<string, unknown>) => {
        value.timestamp = new Date();
        return value;
      }),
    );
  }
}
```

- Code written **before** `return next.handle()` runs before the controller
- Everything piped **after** `next.handle()` runs on the controller's result
- `map()` transforms the response; `tap()` observes without changing it; `catchError()` maps errors

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@UseFilters(ValidationFilter)
@UsePipes(new ValidationPipe(LoginUserRequestValidation))
@Header('Content-Type', 'application/json')
@UseInterceptors(TimeInterceptor)
@Post('/login')
login(
  @Query('name') name: string,
  @Body() // @Body(new ValidationPipe(LoginUserRequestValidation))
  request: LoginUserRequest,
): Record<string, string> {
  return {
    data: `Hello ${request.username}`,
  };
}
```

**Request**

```http
POST http://localhost:8080/api/users/login?name=example
Content-Type: application/json
Accept: application/json

{
  "username": "Dzarurizky",
  "password": "[PASSWORD]"
}
```

**Response**

```http
HTTP/1.1 201 Created
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 66
ETag: W/"42-m6Q2Rzo4Y6gGxrJd2sPxuLa3yhQ"
Date: Sun, 30 Aug 2026 11:21:44 GMT
Connection: close

{
  "data": "Hello Dzarurizky",
  "timestamp": "2026-08-30T11:21:44.035Z"
}
```

Notice the handler returned only `data` — `timestamp` was added by the interceptor. Note also the **201**: that is the default status for POST in NestJS.

> ⚠️ `TimeInterceptor` assumes the handler returns an object. Applied to a route returning a plain string, `value.timestamp = ...` silently does nothing (or throws on `null`). A global version needs a guard: `if (value && typeof value === 'object') { ... }`.

### Global Interceptors

`@UseInterceptors()` on a controller applies to every method in it. To apply one across the whole application, use `app.useGlobalInterceptors()` in `main.ts` — or the `APP_INTERCEPTOR` token if the interceptor needs dependency injection.

---

## 🎨 Custom Decorators

NestJS already provides plenty of decorators for use inside controllers. But sometimes you need your own — for example, when middleware attaches an attribute to the request and you want to read it in the controller *without* dragging in `express.Request`.

Here we build a custom decorator that returns the currently logged-in user.

### Creating a Custom Decorator

Use the `createParamDecorator` function.

**`study-nestjs-basic/src/auth/auth.middleware.ts`**

```ts
import { HttpException, Injectable, NestMiddleware } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service.js';
import type { User } from '../user/user-repository/user-repository.js';
import type { Request, Response } from 'express';

export interface RequestWithUser extends Request {
  user?: User;
}

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private readonly prismaService: PrismaService) {}

  async use(req: RequestWithUser, res: Response, next: () => void) {
    const userId = Number(req.headers['x-user-id']);
    if (!userId) {
      throw new HttpException('Unauthorized', 401);
    }

    const plan = this.prismaService.db.sql.public.users
      .select('id', 'first_name', 'last_name', 'email')
      .where((f, fns) => fns.eq(f.id, userId))
      .build();

    const [user] = await this.prismaService.db.runtime().query(plan);

    if (user) {
      req.user = user;
      next();
    } else {
      throw new HttpException('Unauthorized', 404);
    }
  }
}
```

**`study-nestjs-basic/src/auth/auth.decorator.ts`**

```ts
import { createParamDecorator } from '@nestjs/common';
import type { ExecutionContext } from '@nestjs/common';
import type { RequestWithUser } from './auth.middleware.js';

export const Auth = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest<RequestWithUser>();
    return request.user;
  },
);
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/current')
current(@Auth() user: User): Record<string, string> {
  return {
    data: `Hello ${user.first_name} ${user.last_name}`,
  };
}
```

**`study-nestjs-basic/src/app.module.ts`**

```ts
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LogMiddleware).forRoutes({
      path: '/api/*',
      method: RequestMethod.ALL,
    });
    consumer.apply(AuthMiddleware).forRoutes({
      path: '/api/users/current',
      method: RequestMethod.GET,
    });
  }
}
```

**`study-nestjs-basic/test.http`**

```http
GET http://localhost:8080/api/users/current
x-user-id: 1
```

**Response**

```http
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 28
ETag: W/"1c-0aaStVKfBPUN1pLwWvxkbcjdDHs"
Date: Sun, 30 Aug 2026 12:18:24 GMT
Connection: close

{
  "data": "Hello Dzaru Rizky"
}
```

The `data` parameter of `createParamDecorator` receives whatever is passed at the call site, so `@Auth('first_name')` could return just one field.

> **Tip:** to bundle several decorators into one, use `applyDecorators()` — e.g. combining `@UseGuards(RoleGuard)` and `@Roles([...])` into a single `@RequireRole(...)`.

---

## 🛡️ Guards

Authentication can live in middleware — that's exactly what `AuthMiddleware` above does. But there is a second concern: **authorization**, deciding whether this user is allowed to perform this action.

Middleware is a poor fit for authorization, because middleware doesn't know which route was matched. All it can do is call `next()` — whether that leads to another middleware or the route handler.

NestJS provides **Guards** for this.

### Creating a Guard

Implement the `CanActivate` interface:

```bash
nest generate guard <name> [path]
```

A guard returns `true` (allow) or `false` (deny) — synchronously, as a `Promise`, or as an `Observable`. Returning `false` produces a **403 Forbidden**.

Here we add a `role` column to the `User` model and use a guard to make sure only the listed roles can reach a route.

**`study-nestjs-basic/src/role/role.guard.ts`**

```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';
import type { RequestWithUser } from '../auth/auth.middleware.js';

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private readonly roles: string[]) {}

  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const { user } = context.switchToHttp().getRequest<RequestWithUser>();
    return user != null && this.roles.includes(user.role ?? '');
  }
}
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/current')
@UseGuards(new RoleGuard(['admin', 'operator']))
current(@Auth() user: User): Record<string, string> {
  return {
    data: `Hello ${user.first_name} ${user.last_name}`,
  };
}
```

**Request** — user 2 does not have an allowed role:

```http
GET http://localhost:8080/api/users/current
x-user-id: 2
```

**Response**

```http
HTTP/1.1 403 Forbidden
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 69
ETag: W/"45-MZJWZc+Y+RUbHpnhz2B2Vipii24"
Date: Sun, 30 Aug 2026 12:59:57 GMT
Connection: close

{
  "message": "Forbidden resource",
  "error": "Forbidden",
  "statusCode": 403
}
```

> **Key Insight:** the guard's superpower is `ExecutionContext`. `context.getHandler()` and `context.getClass()` tell it *which* route it's protecting — precisely the information middleware never has. That's what makes route-aware authorization possible, and it's the setup for [Reflector](#-reflector).

---

## ♻️ Lifecycle Events

Every object NestJS manages — modules, controllers, providers — has a **lifecycle**. Hooking into it is useful whenever something must happen at a specific moment during startup or shutdown.

| Hook | When it fires |
| --- | --- |
| `OnModuleInit` | After all modules have been loaded |
| `OnApplicationBootstrap` | After all modules are initialized, before the app accepts connections |
| `OnModuleDestroy` | After a termination signal is received |
| `BeforeApplicationShutdown` | After `OnModuleDestroy`, once all handlers have finished and connections are about to close |
| `OnApplicationShutdown` | After all connections are closed |

Implement the interface and NestJS calls the matching method (`onModuleInit`, `onModuleDestroy`, …).

**`study-nestjs-basic/src/prisma/prisma.service.ts`**

```ts
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { db } from './db.js';

@Injectable()
export class PrismaService implements OnModuleInit, OnModuleDestroy {
  db = db;

  constructor() {
    console.info(`Create prisma service at ${new Date().toISOString()}`);
  }

  async onModuleInit() {
    console.info(`Connect Prisma service at ${new Date().toISOString()}`);
    await this.db.connect();
  }

  async onModuleDestroy() {
    console.info(`Disconnect Prisma service at ${new Date().toISOString()}`);
    await this.db.close();
  }
}
```

**`study-nestjs-basic/src/main.ts`**

```ts
app.enableShutdownHooks();
```

> ⚠️ The three **shutdown** hooks (`OnModuleDestroy`, `BeforeApplicationShutdown`, `OnApplicationShutdown`) only fire if you call `app.enableShutdownHooks()`. It is off by default because it makes NestJS listen on process signals, which carries a small performance cost. Without it, `onModuleDestroy` never runs and the database connection is never closed cleanly.

> **Note:** all lifecycle hooks may be `async`. NestJS awaits them, so `onModuleInit` is the right place to open a connection — the app won't start accepting requests until it resolves.

---

## 🪞 Reflector

We already know how to use and create decorators. **Reflector** is the NestJS utility that makes creating and *reading* metadata decorators easy.

There is a real problem with the `RoleGuard` from the previous section: it requires `new RoleGuard([...])` on every handler. That means one object per route, all of them holding memory, and none of them able to use dependency injection.

The fix is a **singleton** `RoleGuard` plus a helper decorator built with `Reflector`.

**`study-nestjs-basic/src/role/role.decorator.ts`**

```ts
import { Reflector } from '@nestjs/core';

export const Roles = Reflector.createDecorator<string[]>();
```

**`study-nestjs-basic/src/role/role.guard.ts`**

```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';
import type { RequestWithUser } from '../auth/auth.middleware.js';
import { Reflector } from '@nestjs/core';
import { Roles } from './role.decorator.js';

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const roles: string[] = this.reflector.get(Roles, context.getHandler());
    if (!roles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest<RequestWithUser>();
    return user != null && roles.includes(user.role ?? '');
  }
}
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/current')
@Roles(['admin', 'operator'])
@UseGuards(RoleGuard)
current(@Auth() user: User): Record<string, string> {
  return {
    data: `Hello ${user.first_name} ${user.last_name}`,
  };
}
```

Two things changed for the better:

- `RoleGuard` is now a class reference, not an instance — NestJS creates it **once** and can inject `Reflector` into it
- The `if (!roles) return true` branch means routes without `@Roles(...)` pass straight through, so the guard is safe to apply globally

> **Improvement:** `reflector.get(Roles, context.getHandler())` only reads **method-level** metadata, so a `@Roles(...)` placed on the controller class is ignored. Use `getAllAndOverride(Roles, [context.getHandler(), context.getClass()])` to let a method override the class, or `getAllAndMerge(...)` to combine both.

---

## 🌐 Global Providers

Earlier, we made filters, interceptors, guards and pipes global by registering them on the NestJS application. That approach has a problem: creating the object manually with `new` means it **cannot use dependency injection**.

### Provider Aliases

NestJS provides alias tokens for exactly this. Register the class as an ordinary provider, but under a name NestJS recognizes:

| Token | Registers a global… |
| --- | --- |
| `APP_GUARD` | Guard |
| `APP_INTERCEPTOR` | Interceptor |
| `APP_PIPE` | Pipe |
| `APP_FILTER` | Exception filter |

**`study-nestjs-basic/src/app.module.ts`**

```ts
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller.js';
import { AppService } from './app.service.js';
import { UserModule } from './user/user.module.js';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { ValidationModule } from './validation/validation.module.js';
import { LogMiddleware } from './log/log.middleware.js';
import { AuthMiddleware } from './auth/auth.middleware.js';
import { APP_GUARD } from '@nestjs/core';
import { RoleGuard } from './role/role.guard.js';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
    ValidationModule.forRoot(true),
  ],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: APP_GUARD,
      useClass: RoleGuard,
    },
  ],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LogMiddleware).forRoutes({
      path: '/api/*',
      method: RequestMethod.ALL,
    });
    consumer.apply(AuthMiddleware).forRoutes({
      path: '/api/users/current',
      method: RequestMethod.GET,
    });
  }
}
```

**`study-nestjs-basic/src/user/user/user.controller.ts`**

```ts
@Get('/current')
@Roles(['admin', 'operator'])
current(@Auth() user: User): Record<string, string> {
  return {
    data: `Hello ${user.first_name} ${user.last_name}`,
  };
}
```

`@UseGuards(RoleGuard)` is gone — the guard now runs on every route in the application, and `@Roles(...)` is all that's left to declare.

> **Note:** the same token can be registered more than once — several `APP_INTERCEPTOR` entries all take effect. And unlike `app.useGlobalX()`, providers registered this way live in the module's injector, so they can inject anything the module can see.

---

## 🔁 Request Lifecycle

Knowing the order in which the pieces run is what makes the whole framework click. For every incoming request, NestJS executes:

```
Request
  ↓
Middleware            global → module-bound
  ↓
Guards                global → controller → route
  ↓
Interceptors (pre)    global → controller → route
  ↓
Pipes                 global → controller → route → parameter
  ↓
Controller (handler)  ← your method runs here
  ↓
Interceptors (post)   route → controller → global   (reverse order)
  ↓
Exception Filters     route → controller → global   (only if something threw)
  ↓
Response
```

This explains the design decisions throughout this guide:

| Question | Answer |
| --- | --- |
| Why can't middleware do authorization? | It runs before NestJS resolves which handler was matched, so it has no route metadata |
| Why does a guard see `@Roles(...)` but middleware doesn't? | Guards receive an `ExecutionContext` with `getHandler()` and `getClass()` |
| Why do pipes run after guards? | No point parsing and validating a payload for a request that's about to be rejected |
| Why can an interceptor modify the response? | It wraps the handler on both sides, so it observes the returned value |
| Why do exception filters come last? | They are the safety net catching anything thrown anywhere above them |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Module** | Group related controllers and providers | `@Module({ imports, controllers, providers, exports })` |
| **Controller** | Handle HTTP requests for a path prefix | `@Controller('/api/users')` |
| **Route** | Bind a method to an HTTP method + path | `@Get('/:id')`, `@Post()` |
| **Request data** | Read query/param/body/headers | `@Query()`, `@Param()`, `@Body()`, `@Headers()` |
| **Response shape** | Set status code and headers | `@HttpCode(200)`, `@Header(k, v)`, `@Redirect()` |
| **Provider** | An injectable class (service, repository…) | `@Injectable()` + register in `providers` |
| **Dependency Injection** | Receive a provider automatically | `constructor(private service: UserService) {}` |
| **Custom Provider** | Control how a dependency is built | `useClass`, `useValue`, `useFactory`, `useExisting` |
| **ModuleRef** | Fetch a provider manually at runtime | `this.moduleRef.get(Connection)` |
| **Config** | Read `.env` values | `ConfigModule.forRoot({ isGlobal: true })` + `ConfigService` |
| **Global Module** | Auto-import a module everywhere | `@Global()` on the module class |
| **Dynamic Module** | Build a module from runtime options | `static forRoot(): DynamicModule` |
| **Middleware** | Pre-processing, before routing is known | `implements NestMiddleware` + `configure(consumer)` |
| **Guard** | Allow or deny access to a route | `implements CanActivate` + `@UseGuards()` |
| **Pipe** | Transform / validate a parameter | `implements PipeTransform` + `@UsePipes()` |
| **Interceptor** | Wrap the handler, transform the response | `implements NestInterceptor` + `@UseInterceptors()` |
| **Exception Filter** | Turn an error into an HTTP response | `@Catch(ErrorType)` + `@UseFilters()` |
| **Custom Decorator** | Extract data from the request | `createParamDecorator((data, ctx) => …)` |
| **Reflector** | Create and read metadata decorators | `Reflector.createDecorator<T>()` |
| **Global Provider** | Global filter/guard/pipe/interceptor with DI | `{ provide: APP_GUARD, useClass: RoleGuard }` |
| **Lifecycle Hook** | Run code on startup / shutdown | `implements OnModuleInit, OnModuleDestroy` |

---

## 💡 Best Practices

**✅ Do This**

- **Prefer `@Query()`, `@Param()` and `@Body()` over `@Req()`** — the handler declares exactly what it needs, stays decoupled from Express, and unit tests become plain function calls with no mocks
- **Return a value instead of using `@Res()`** — reach for the raw response only for genuinely Express-specific work like `render()` or streaming
- **Declare specific routes before dynamic ones** — `@Get('/sample')` must come before `@Get('/:id')`, or it becomes unreachable
- **Use constructor injection** and keep dependencies visible in the signature
- **Export only what other modules genuinely need** — `exports` is a module's public API, and everything else stays a private implementation detail
- **Register global filters, guards, pipes and interceptors via `APP_FILTER` / `APP_GUARD` / `APP_PIPE` / `APP_INTERCEPTOR`** so they can still use dependency injection
- **Give a global guard an escape hatch** — `if (!roles) return true` keeps unannotated routes working
- **Call `app.enableShutdownHooks()`** whenever a provider implements a shutdown lifecycle hook, or `onModuleDestroy` silently never runs
- **Close the app in `afterEach`** of every E2E test (`await app.close()`), otherwise Jest hangs on open handles
- **Use `getAllAndOverride()` in a Reflector-based guard** so class-level and method-level metadata both work

**❌ Avoid This**

- **Confusing `@Headers()` with `@Header()`** — the plural one reads a *request* header, the singular one sets a *response* header
- **Injecting `@Res()` and still returning a value** — the return value is ignored and the request hangs until it times out
- **Writing bare `*` wildcards in NestJS 11** — Express 5 requires named wildcards (`'/api/*path'`), and the old form quietly stops matching
- **Using `__dirname` in an ESM project** — with `"type": "module"` it doesn't exist; rebuild it from `import.meta.url`
- **Naming a custom pipe `ValidationPipe`** — it collides with the one exported by `@nestjs/common`
- **Marking every module `@Global()`** — `imports` is what documents a module's dependencies; reserve `@Global()` for real infrastructure like config, logging, and the database
- **Reaching for `ModuleRef` instead of the constructor** — it hides dependencies and undoes the point of dependency injection
- **Writing an interceptor that assumes the handler returned an object** — guard with `typeof value === 'object'` before mutating it, especially when the interceptor is global
- **Mutating state in a `@Get()` route** — GET is meant to be safe, and is freely cached and retried
- **Registering a provider a controller needs without listing it in the test module** — `Test.createTestingModule` builds a fresh injector and won't find it

> Reference: [docs.nestjs.com](https://docs.nestjs.com/) · [github.com/nestjs/nest](https://github.com/nestjs/nest)
