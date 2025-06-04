---
{"dg-publish":true,"permalink":"//next-js/prisma/"}
---


Prisma는 데이터베이스와 상호작용하기 위해 클라이언트와 서버 간의 커뮤니케이션을 처리하는 클라이언트 라이브러리와 데이터베이스 스키마를 정의하고 관리하는 마이그레이션 관리 도구로 구성되있다.

Prisma 클라이언트는 Prisma 스키마에서 정의된 데이터 모델과 일치하는 코드를 생성한다. 이 코드는 TypeScript로 작성되며, 데이터베이스와 상호작용할 때 타입 안정성을 보장한다.

Prisma 서버는 Prisma 클라이언트와 데이터베이스 사이에서 요청을 중개하고, Prisma 스키마를 기반으로 클라이언트가 데이터를 가져오거나 수정하는 방식을 결정한다.


#### schema.prisma 경로 재설정
---
```json{

//package.json
 "prisma": {
    "schema": "./src/server/db/schema.prisma"
  }
 }
```


#### Connection pool ,Pool time out설정
---
```
datasource db { provider = "postgresql" url = "postgresql://johndoe:mypassword@localhost:5432/mydb?connection_limit=5&pool_timeout="20"
}
```
 - connection_limit : 서버리스 환겨에서는 1
 - pool_timeout : 기본값 10 , 대기열 사용 0 

#### pgBouncer 사용 가능

#### Log 확인
---
```ts
// prismaClient.ts

import { PrismaClient } from "@prisma/client";

class Database {
  private static instance: PrismaClient;

  private constructor() {}
  static getInstance(): PrismaClient {
    if (!Database.instance) {
      Database.instance = new PrismaClient({
        log: ["query", "info", "warn", "error"],
      });
    }
    return Database.instance;}}

export default Database.getInstance();
```

#### 트랜잭션
---

프리스마에서는 트랜잭션을 4가지 형태로 지원한다.
1. 중첩쓰기 : 실행하는 쿼리에 해당하는 레코드와 관련된 여러 작업 수행
2. 대량 트랜잭션 : 
    - `deleteMany`
    - `updateMany`
    - `createMany`
      
3. 대화형 트랜잭션 
4. 순차적 트랜잭션 : [] 에 들어간 쿼리 순으로 작업 수행


####  $transaction API
---
prisma 트랜잭션에서 사용가능한 격리 수준은 아래와 같다. default 는 사용하는 Db의 기본값이 적용된다.
- `ReadUncommitted`
- `ReadCommitted`
- `RepeatableRead`
- `Snapshot`
- `Serializable`

```ts
try {
await prisma.$transaction(async (tx) => {
// Code running in a transaction...
},{
// maxWait, timeOut 은 대화형 트랜잭션에서만 설정가능
maxWait: 5000, // default: 2000 
timeout: 10000, // default: 5000
isolationLevel: Prisma.TransactionIsolationLevel.Serializable, 
// defalut : db 기본
})

} catch (err) {
// Handle the rollback...
}
```



 **대화형 트랜잭션**

```ts
describe("* 프리스마 테스트", () => {
  it("대화형 트랜잭션", async () => {
    const interactiveTx = await prisma.$transaction(async (tx) => {
      const countBefore = await prisma.user.count();
      //  테스트 값 생성
      const randomEmail = Math.random() + "";
      await prisma.user.create({
        data: {
          email: randomEmail,
          password: "test",
          name: "test",
        },
      });
      const countAfter = await prisma.user.count();
      
      return [countBefore, countAfter];
    });
    expect(interactiveTx[0] + 1).toBe(interactiveTx[1]);
  });
});
```



**순차적 트랜잭션**

아래와 같이 대량작업을 트랜잭션 하나로 관리 가능하다.
```ts
const id = 9 // User to be deleted 
const deletePosts = prisma.post.deleteMany({
	where: { userId: id, }, }) 
const deleteMessages = prisma.privateMessage.deleteMany({ 
	where: { userId: id, }, }) 
const deleteUser = prisma.user.delete({ 
	where: { id: id, }, }) 
	
await prisma.$transaction([deletePosts,deleteMessages,deleteUser]) // Operations succeed or fail together
```


#### 미들웨어
```ts
import { PrismaClient } from "@prisma/client";

  
const globalForPrisma = global as unknown as {
  prisma: PrismaClient | undefined;
};


export const prisma =
  globalForPrisma.prisma ?? new PrismaClient({ log: ["query"] });


// log 미들웨어 
// params : prisma.user.create ~~~ 의 경우 
// user: prams.model
// create : params.action : 
prisma.$use(async (params, next) => {
  const before = Date.now();
  const result = await next(params);
  const after = Date.now();
  console.log(
    `Query ${params.model}.${params.action} took ${after - before}ms`
  );
  return result;
});

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```
https://www.prisma.io/docs/orm/prisma-client/client-extensions/middleware/soft-delete-middleware