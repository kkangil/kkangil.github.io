---
title: deno oak framework
toc: true
widgets:
  - type: toc
    position: right
  - type: categories
    position: right
  - type: tags
    position: right
  - type: adsense
    position: right
    client_id: ca-pub-5445993070474035
    slot_id: ''
date: 2020-07-05 11:20:34
categories:
    - Deno
tags:
    - Deno REST API
    - oak
    - oak middleware
---

Deno 의 framework 를 사용하여 API 서버를 구축해보려한다. 현재 기준(2020/07/05) 수 많은 framework 가 있지만
oak 가 제일 인기 많고 잘 만들어져 있다고 한다. 사용법이 express 와 비슷해서 인기가 많아진 것 같지만
oak 에서 제공해주는 example 을 직접 구현해 보려 한다. 

<!-- more -->

## server

```typescript
import {
  Application,
} from "https://deno.land/x/oak/mod.ts";

const app = new Application();

console.log(`Server is listening on port 8000`);
await app.listen({ port: 8000 });
```

oak 에서 제공해주는 Application 을 사용하여 서버를 실행한다. 이때 listen 함수는 promise 기 때문에
await 을 붙여줘서 실행하면 된다.

```shell script
deno run --allow-net server.ts
```

이렇게 서버를 실행해보면 에러가 발생한다. 이 에러는 현재 request 에 대한 처리나 route 가 없어서 발생하는
에러이므로 일단 다음 단계로 넘어가서 추가해주면 된다.

## routes

```typescript
import {
  Router,
} from "https://deno.land/x/oak/mod.ts";

const router = new Router();

app.use(router.routes());
app.use(router.allowedMethods());

router.get("/", (context) => {
  context.response.body = "Hello deno, oak";
});

```

oak 에서 Router 를 가져와 router 를 생성해준다. 이후 app middleware 에 등록해 주는것이다.
이때 app.use 를 사용해서 등록하는데 express 와 동일하다.

router.routes() 는 router 의 path 를 등록해주는 것이고, allowedMethods() 는 해당 path 의 모든 http method 를
허용해 주는 것이다.

이후 express 의 router 와 동일하게 router.get 함수를 이용하여 router 를 등록해 줄 수 있다.
차이점은 두번째 parameter 함수다. 기존에는 (req, res, next) 형식의 parameter 를 사용했었는데, oak 는 이것을
context 하나의 객체로 사용한다. destructuring 을 사용하여 쓸 수도 있어 더 편해진것 같다.

이제 다시 서버를 시작해보면 에러가 사라지고 8000번에 접속해보면 정상 작동하는것을 확인할 수 있다.

route 와 server 분리를 위해 routes.ts 파일을 생성해준다.

```typescript
import {
  Router,
} from "https://deno.land/x/oak/mod.ts";

const router = new Router();

router.get("/", (context) => {
  context.response.body = "Hello deno, oak";
});

export default router;
```

이후 server.ts 에서 해당 router 를 import 해준다.

```typescript
import {
  Application,
} from "https://deno.land/x/oak/mod.ts";

import router from "./routes.ts";

const app = new Application();

app.use(router.routes());
app.use(router.allowedMethods());

console.log(`Server is listening on port 8000`);
await app.listen({ port: 8000 });
```

## route method

```typescript
interface Book {
  id: string;
  title: string;
  author: string;
}

let books: Book[] = [
  {
    id: "1",
    title: "Book 1",
    author: "one",
  },
  {
    id: "2",
    title: "Book 2",
    author: "two",
  },
  {
    id: "3",
    title: "Book 3",
    author: "three",
  },
];
```

위 interface 구조를 가진 책 데이터를 사용하여 진행하려고 한다.

```typescript
router
    .get("/", ({ response }) => {
        response.body = "Hello deno, oak";
    })
    .get("/books", ({ response }) => {
        response.body = books;
    })
    .get("/books/:id", ({ params, response }) => {
        const book: Book | undefined = books.find((book) => book.id === params.id);
        if (book) {
          response.body = book;
        } else {
          response.body = "존재하지 않는 책";
          response.status = 404;
        }
    });
```

`/books` 로 get 요청이 들어오면 전체 책을 return 해준다. body 에 담아주면 된다.

`/books/:id` 로 get 요청이 들어오면 params 의 id 에 맞는 책을 찾아 return 해준다.
이때 express 와 다른점은 request 에 params 정보가 들어있는 것이 아니라 context 에서 따로 제공해주기 때문에
request 를 사용할 필요가 없다. 

id 에 맞는 책을 찾아 리턴해주는데 존재하지 않은 책이면 status 를 404 로 리턴해준다.

### post
```typescript
.post("/books", async ({ request, response }) => {
    const body = await request.body();

    if (!request.hasBody) {
      response.status = 400;
      response.body = "데이터 없음";
    } else {
      const book: Book = body.value;
      book.id = v4.generate();
      books.push(book);
      response.status = 201;
      response.body = book;
    }
  })
```

express 에서는 body-parser 를 사용하여 request.body 로 body 데이터를 받아오지만, oak 는 request.body promise
로 제공해준다. 또한 request.hasBody 를 사용하여 body 데이터가 있는지 확인할 수 있다.

DB 를 사용하는 경우 id 생성을 자동으로 해줄 수 있지만 지금은 std 의 uuid 를 사용하여 id 를 생성해줬다.

postman 으로 테스트해보면 정상 동작 하는것을 확인할 수 있다.

### put

```typescript
.put("/books/:id", async ({ params, request, response }) => {
    const bookIndex: number = books.findIndex((book) => book.id === params.id);
    if (bookIndex < 0) {
      response.body = "존재하지 않는 책";
      response.status = 404;
    } else {
      if (!request.hasBody) {
        response.status = 400;
        response.body = "데이터 없음";
      } else {
        const body = await request.body();
        const book: Book = body.value;
        const preBook: Book = books[bookIndex];
        books.splice(bookIndex, 1, { ...preBook, ...book });
        response.status = 201;
        response.body = books[bookIndex];
      }
    }
  })
```

Array.findIndex 를 사용하여 책을 찾아주고 수정해 주었다.

### delete

```typescript
.delete("/books/:id", async ({ params, response }) => {
    const bookIndex: number = books.findIndex((book) => book.id === params.id);
    if (bookIndex < 0) {
      response.body = "존재하지 않는 책";
      response.status = 404;
    } else {
      books.splice(bookIndex, 1);
      response.status = 200;
    }
  })
```

delete method 도 express 와 동일한 방식으로 사용가능 하다.

## middleware

### logger

```typescript
import {
  green,
  cyan,
  bold,
} from "https://deno.land/std@0.60.0/fmt/colors.ts";
import {
  Context,
} from "https://deno.land/x/oak/mod.ts";

const logger = async (context: Context, next: () => Promise<void>) => {
  await next();
  const rt = context.response.headers.get("X-Response-Time");
  console.log(
    `${green(context.request.method)} ${
      cyan(decodeURIComponent(context.request.url.pathname))
    } - ${
      bold(
        String(rt),
      )
    }`,
  );
};

export default logger;
```

```typescript
import {
  Context,
} from "https://deno.land/x/oak/mod.ts";

const responseTime = async (context: Context, next: () => Promise<void>) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  context.response.headers.set("X-Response-Time", `${ms}ms`);
};

export default responseTime;
```

console 에 기록을 남기는 Logger 함수와 응답시간을 추적하는 ResponseTime 함수를 생성해 준 후 
express middleware 등록과 같이 app.use 를 사용해서 등록해준다.

```typescript
app.use(logger);
app.use(responseTime);
```

### 404 (not found)

```typescript
import {
  Status,
  Context,
} from "https://deno.land/x/oak/mod.ts";

const notFound = ({ request, response }: Context) => {
  response.status = Status.NotFound;
  response.body =
    `<html><body><h1>404 - Not Found</h1><p>Path <code>${request.url}</code> not found.`;
};

export default notFound;
```

존재하지 않는 페이지에 접근했을때 실행되는 함수이다. route middleware 밑에 등록해주면 된다.

```typescript
app.use(router.routes());
app.use(router.allowedMethods());
// A basic 404 page
app.use(notFound);
```

### error handler

```typescript
import {
  isHttpError,
  Context,
} from "https://deno.land/x/oak/mod.ts";

const errorHandler = async (
  { request, response }: Context,
  next: () => Promise<void>,
) => {
  try {
    await next();
  } catch (err) {
    if (isHttpError(err)) {
      response.status = err.status;
      const { message, status, stack } = err;
      if (request.accepts("json")) {
        response.body = { message, status, stack };
        response.type = "json";
      } else {
        response.body = `${status} ${message}\n\n${stack ?? ""}`;
        response.type = "text/plain";
      }
    } else {
      console.log(err);
      throw err;
    }
  }
};

export default errorHandler;

```

errorHandler 함수를 만들어서 middleware 에 등록시켜준다.
추가로 router context 에 throw 라는 함수를 제공해준다. 이 함수를 실행하여 에러를 발생 시킬 수 있다.

```typescript
context.throw(Status.NotFound, "존재하지 않는 책");
```

[Github code 보기](https://github.com/kkangil/deno-oak-app)
[oak routing server](https://deno.land/x/oak/examples/routingServer.ts)
