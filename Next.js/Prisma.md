# Prisma 설치 방법
1. 다음 입력
	```
	npm i -D prisma
	npm i @prisma/client
	```

2. `db.ts` 생성하기: 스프링부트의 application.yml과 비슷한 역할입니다.
	```ts
	import { PrismaClient } from "@prisma/client"
	  
	//development
	declare global {
	  var prisma: PrismaClient | undefined
	}
	  
	export const db = globalThis.prisma || new PrismaClient()
	  
	if (process.env.NODE_ENV !== "production") globalThis.prisma = db
	  
	//production
	// export const db = new PrismaClient()
	```

3. `.gitignore`에 `.env` 추가하기: `.env`에 prisma 설정 값을 작성하기 때문에 github에 올라가면 안됩니다. 

4. `npx prisma init` 실행하기
	- 프로젝트 폴더 최상위에 `.env`파일이 생성됩니다.
	- `DATABASE_URL`에 실제 데이터베이스 URL을 할당해야 합니다.

5. 서버리스 postgresDB를 제공하는 neon에서 서버리스 데이베이스를 만듭니다.
	1. [neon](https://neon.tech/) 로그인하기
	2. start free > Projects > New Project > 프로젝트 명, 데이터이스 명 입력, 지역에따라 Region 선택(싱가포르 선택) > Create project 클릭
	3. CONNECTION DETAILS FOR YOUR NEW PROJECT > 드롭다운 Prisma 선택
	4. schema.prisma 복사하고 `prisma` > `schema.prisma`에 기존 내용 지우고 붙여넣기
	5. .env 복사하고 `.env`에 기존 내용 지우고 붙여넣기

6. `schema.prisma`에 다음을 작성해서 모델이 생성될 수 있도록 합니다.
	```
	generator client {
	  provider = "prisma-client-js"
	}
	```

7. `schema.prisma`에 모델을 작성합니다. JPA의 엔티티와 비슷한 개념으로, ORM객체를 뜻합니다.
	```
	model User {
	  id String @id @default(cuid())
	  name String
	}
	```

8. 작성한 모델을 토대로 데이터베이스에 테이블을 만듭니다. 
> [!tip] 모델을 추가할때마다 실행해야 데이터베이스에 테이블이 생성됩니다. 하지만 아직 서버리스 데이터베이스에 올라간 상태는 아닙니다.

```
npx prisma generate
```

9. 서버리스 데이터베이스에 생성한 테이블 동기화하기
```
npx prisma db push

>>>Your database is now in sync with your Prisma schema. Done in 37.11s
```

# Prisma 모델 사용하기
- 컴포넌트 등의 파일에서 prisma로 생성한 모델 객체를 사용합니다.
	```tsx
	import { db } from "@/lib/db"
	
	export default async function jMyComponent({
	  children,
	}: Readonly<{
	  children: React.ReactNode
	}>) {
	  const user = await db.user.findFirst()
	  
	  return (
	    //...
	  )
	}
	```

	![[prisma-model-usecase.png]]

# 데이터베이스에 데이터 넣기
- 다음을 실행하면 우리가 설정한 `prisma.schema`가 neon에 만들어둔 서버리스 데이터베이스와 동기화됩니다.
```
npx prisma generate
npx prisma db push
```

- neon에서 BRANCH > Tables에 들어가면 만들어둔 모델이 테이블로 생성된 걸 확인할 수 있습니다.
![[prisma-table.png]]
# 데이터베이스 초기화하기
```
npx prisma migrate reset
```
# Prisma에 저장된 데이터 보는 법
- 다음 명령을 실행하면 localhost:5555가 열리며 Prisma에 저장된 데이터를 볼 수 있습니다.
```
npx prisma studio
```