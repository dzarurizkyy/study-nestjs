# Study NestJS 🪺

This repository contains a comprehensive reference guide for NestJS — covering the fundamentals (modules, controllers, providers, dependency injection), the full request-handling pipeline (middleware, guards, interceptors, pipes, exception filters), database access with Prisma, and a complete Contact Management REST API built end-to-end and covered by e2e tests.

## List of Material 📚

- 🪺 **[NestJS Basics](001-nestjs-basics.md)**

  Modules, controllers, providers, and dependency injection, built hands-on inside a `study-nestjs-basic` playground — through to the whole request pipeline: middleware, pipes, guards, interceptors, exception filters, custom decorators, and lifecycle events:

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

- 🚀 **[NestJS RESTful API Study Case — Contact Management](002-nestjs-studycase.md)**

  A complete REST API built stage by stage across three resources — User → Contact → Address — with token authentication, Zod validation at the service layer, Prisma 7 + PostgreSQL, and 66 Jest + Supertest e2e tests, plus a full debugging journal of every real bug hit along the way:

  ```ts
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

## 📍 References

- [Udemy](https://www.udemy.com/course/belajar-nestjs/?srsltid=AfmBOoqhnIFajApZCjytQM3PdcKJiFKxb81b8Q5eoXriPMiOsHANJs7N)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
