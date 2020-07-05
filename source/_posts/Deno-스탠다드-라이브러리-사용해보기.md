---
title: Deno 스탠다드 라이브러리 사용해보기
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
date: 2020-07-05 09:58:27
categories:
    - Deno
tags:
    - Deno std library
---

최근 Node 의 단점들을 보완한 Deno 가 정식 릴리즈 되었다. 아직 1 버전 으로 실제 회사나 프로젝트에서 쓰기는
무리가 있겠지만, 버전이 올라가면서 안정적으로 되면 Node 를 대체할 수도 있을거라고 생각한다.

Deno 가 왜 좋은지, 어떤 장점과 이점이 있는지는 검색해보면 많이 있다. 나는 이런 이론 말고 직접 한번 써보고 싶었다.

<!-- more -->

Deno 는 `standard Library(이하 std)` 라고 직접 몇몇 기능을 가진 함수들을 제공한다. 예를 들면 date 관련 utils, 
uuid 관련 utils 함수를 제공해준다. 오늘은 이 std 기능들 중 유용한 것들을 직접 구현해보려 한다.

들어가기 앞서 Deno 는 기본적으로 ts 를 쓸 수 있으며 `top-level-await` 를 제공해 주므로 async 없이 await 쓰는걸
많이 볼 것이다.

[top-level-await 참고](https://github.com/tc39/proposal-top-level-await)

## Permission

Node 와 비교하여 가장 큰 특징으로 꼽히는 것이 바로 이 Permission 이다. Node 는 파일 실행시 모든 권한이 허용 되어
있다. 하지만 Deno 는 상황에 맞는 권한을 항상 허용해 주어야 한다. deno 는 deno run [flag] [파일명] 으로 실행한다.

|  flag           | description                      |
|-----------------|----------------------------------|
| --allow-read    | 읽기 권한 허용                   |
| --allow-write   | 쓰기 권한 허용                   |
| --allow-net     | network 권한 허용(ex. port 열기) |
| --allow-env     | 환경 접근 권한 허용              |
| --allow-run     | 실행 중인 하위 프로세스 허용     |
| --allow-all(-A) | 모든 권한 허용                   |

## file 생성, 복사

std 말고 Deno 가 기본적으로 내장하고 있는 기능이다.

```typescript
const encoder = new TextEncoder();
const text = encoder.encode("hello deno!");

await Deno.writeFile("hello.txt", text);
```

- Deno 의 writeFile 을 사용하여 file 을 생성한다.
- Deno 함수는 기본적으로 promise 로 이루어져 있다.

```shell script
deno run --allow-write createFile.ts
```

```typescript
let file = await Deno.open("hello.txt");
await Deno.copy(file, Deno.stdout);
file.close();
```

- 위에서 생성한 file 을 복사한다. 실행해보면 console 에 hello deno! 가 찍힌다.

```shell script
deno run --allow-read copyFile.ts
```

## archive

archive 는 `tar` 와 `untar` 기능을 갖고있다.

### tar

```typescript
import { Tar } from "https://deno.land/std/archive/tar.ts";

const tar = new Tar();
const content = new TextEncoder().encode("Deno.land");
await tar.append("deno.txt", {
  reader: new Deno.Buffer(content),
  contentSize: content.byteLength,
});

const writer = await Deno.open("./out.tar", { write: true, create: true });
await Deno.copy(tar.getReader(), writer);
writer.close();
```

- Deno.land 라는 text 파일을 생성 후 같은 경로에 이 파일을 포함하고 있는 out.tar 를 생성한다.
- writer open 을 한 후 마지막에 close 를 해줘야한다.
- 위 코드에서는 파일을 생성해 주기 때문에 `--allow-write` 를 해주어야 한다.
```shell script
deno run --allow-write tar.ts
```

### untar

```typescript
import { Untar } from "https://deno.land/std/archive/tar.ts";
import { ensureFile } from "https://deno.land/std/fs/ensure_file.ts";
import { ensureDir } from "https://deno.land/std/fs/ensure_dir.ts";

const reader = await Deno.open("./out.tar", { read: true });
const untar = new Untar(reader);

for await (const entry of untar) {
  console.log(entry); // metadata
  /*
    TarEntry {
      fileName: "deno.txt",
      fileMode: 511,
      mtime: 1593911999,
      uid: 0,
      gid: 0,
      type: "file",
      fileSize: 9
    }
  */

  if (entry.type === "directory") {
    await ensureDir(entry.fileName);
    continue;
  }

  await ensureFile(entry.fileName);
  const file = await Deno.open(entry.fileName, { write: true });
  // <entry> is a reader
  await Deno.copy(entry, file);
}
reader.close();

```

- tar.ts 에서 생성한 out.tar 를 압축 해제하는 기능이다.
- untar 과정은 out.tar untar -> 파일들을 읽어옴 -> 파일을 같은 경로에 생성 순으로 이루어 진다. 이 순으로 인해
읽기와 쓰기 권한을 허용해 주어야 한다.

```shell script
deno run --allow-read --allow-write untar.ts
```

## datetime

string data 를 Date 객체로 변환해주는 간단한 날짜 관련 기능이다.

```typescript
import {
  parseDate,
  parseDateTime,
  dayOfYear,
  currentDayOfYear,
} from "https://deno.land/std/datetime/mod.ts";

console.log(parseDate("20-01-2019", "dd-mm-yyyy")); // output : new Date(2019, 0, 20)
console.log(parseDate("2019-01-20", "yyyy-mm-dd")); // output : new Date(2019, 0, 20)

console.log(parseDateTime("01-20-2019 16:34", "mm-dd-yyyy hh:mm")); // output : new Date(2019, 0, 20, 16, 34)
console.log(parseDateTime("16:34 01-20-2019", "hh:mm mm-dd-yyyy")); // output : new Date(2019, 0, 20, 16, 34)

console.log(dayOfYear(new Date("2020-05-05T10:24:00"))); // output: 126
console.log(currentDayOfYear()); // output: ** depends on when you run it :) **

```

- parseDate: 날짜까지만 Date 객체로 변환한다. (DateFormat)
- parseDateTime: 시간까지 Date 객체로 변환한다. (DateTimeFormat)
- dayOfYear: 1월 1일을 기준으로 넘겨준 날짜가 몇일이 지났는지 리턴해준다.
- currentDayOfYear: 1월 1일을 기준으로 현재 몇일이 지났는지 리턴해준다.

## http

port 를 사용하여 server 를 실행시켜주거나, cookie 등 http 와 관련된 함수들을 제공한다.

### serve

```typescript
import { serve } from "https://deno.land/std/http/server.ts";
const server = serve({ port: 8000 });

console.log("http://localhost:8000/");

for await (const req of server) {
  req.respond({ body: "Hello World\n" });
}
```

```shell script
deno run --allow-net serve.ts
```

- serve 함수를 실행하여 server 를 시작한다. 현재는 option 으로 port 만 넘겨주었지만, hostname, HTTPS 를 위한 
certFile, keyFile 도 제공해 준다.
- server 의 req 를 잡아와 respond 함수를 실행해 줌으로써 8000 번 port 에 접속해보면 해당 문구가 찍히는 것을
확인할 수 있다.

## UUID

uuid (v1, v4, v5) 생성 함수를 제공한다.

```typescript
import { v4 } from "https://deno.land/std/uuid/mod.ts";

// Generate a v4 uuid
const myUUID = v4.generate();
console.log(myUUID);

// Validate a v4 uuid
const isValid = v4.validate(myUUID);
console.log(isValid); // output: true

```

```shell script
deno run uuid.ts
```

## ws

웹 소켓 기능도 제공해준다. std 의 코드 양이 많으므로 std 에서 제공해주는 코드를 실행해보는것으로 대체해 보려한다.
deno 는 현재 경로의 파일 뿐만 아니라 http 요청을 통해서도 실행이 가능하다.

우선 ws server 를 실행시켜보자.

```shell script
deno run --allow-net https://deno.land/std/ws/example_server.ts
``` 

실행시켜보면 socket connected! 라는 문구가 뜨면 성공이다. 이 후 터미널을 새로 열어 client server 를 실행해준다.

```shell script
deno run --allow-net https://deno.land/std/ws/example_client.ts
```

ws connected! 가 뜨면 성공이다. 이제 client 에서 text 를 입력하고 엔터를 눌러보면 server console 에 해당 문구가 
찍히는것을 확인할 수 있다.

## conclusion

위의 스탠다드 라이브러리는 deno 에서 제공해주는것에 일부분에 불과하다. 자주 사용될 것만 우선 정리해둔 것이다.
직접 공식문서를 확인해 보는것을 추천한다.
