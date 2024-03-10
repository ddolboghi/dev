# Prisma 설치 방법
1. 다음 입력
	```
	npm i -D prisma
	npm i @prisma/client
	```

2. `db.ts` 생성하기
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
4. `npx prisma init`
 실행하기: 프로젝트 폴더 최상위에 `.env`파일이 생성됩니다.
5. 