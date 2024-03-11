- next auth의 가장 중요한 목적은 OAuth입니다.
- 로그인 후 리다이렉션되는 페이지를 만들어야합니다.
- [초기 설정](https://next-auth.js.org/configuration/initialization)
- 리다이렉트 주소: `/api/auth/callback/:provider`
# [공식문서](https://authjs.dev/guides/upgrade-to-v5)

# 로그인, 회원가입 폼 만들기
## zod를 이용한 validation
- `index.ts`: `zod`를 이용해 검증 schema를 작성합니다. 
```ts
// lib/index.ts
import * as z from "zod"
  
export const LoginSchema = z.object({
  email: z.string().email({
    message: "이메일은 필수 입력 항목입니다.",
  }),
  password: z.string().min(1, {
    message: "비밀번호는 필수 입력 항목입니다.",
  }),
})
  
export const RegisterSchema = z.object({
  email: z.string().email({
    message: "이메일은 필수 입력 항목입니다.",
  }),
  password: z.string().min(6, {
    message: "비밀번호는 6자리 이상이어야 합니다.",
  }),
  name: z.string().min(1, {
    message: "이름은 필수 입력 항목입니다.",
  }),
})
```

- 회원가입 폼 컴포넌트를 작성합니다.
```tsx
// components/auth/registerForm.tsx
"use client"
  
import * as z from "zod"
import { CardWrapper } from "./cardWrapper"
import { zodResolver } from "@hookform/resolvers/zod"
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "../ui/form"
import { RegisterSchema } from "@/schemas"
import { useForm } from "react-hook-form"
import { Input } from "../ui/input"
import { Button } from "../ui/button"
import FormError from "../formError"
import FormSuccess from "../formSuccess"
import { register } from "@/actions/register"
import { useState, useTransition } from "react"

export function RegisterForm() {
const [error, setError] = useState<string | undefined>("")
  const [success, setSuccess] = useState<string | undefined>("")
  const [isPending, startTransition] = useTransition() //대기상태인지 여부 설정
  
  const form = useForm<z.infer<typeof RegisterSchema>>({
    resolver: zodResolver(RegisterSchema),
    defaultValues: {
      email: "",
      password: "",
      name: "",
    },
  })
  
  const onSubmit = (values: z.infer<typeof RegisterSchema>) => {
    setError("")
    setSuccess("")
  
    startTransition(() => {
      register(values).then((data) => {
        setError(data.error)
        setSuccess(data.success)
      })
    })
  }
  return (
    //...
  )
}
```

- `register.ts`: 사용자가 입력한 값을 받아 처리하는 곳입니다.
```ts
// actions/register.ts
"use server"
  
import * as z from "zod"
import { RegisterSchema } from "@/schemas"
  
export const register = async (values: z.infer<typeof RegisterSchema>) => {
  const validateFields = RegisterSchema.safeParse(values)
  
  if (!validateFields.success) {
    return { error: "Invalid fields!" }
  }
  
  return { success: "Email sent!" }
}
```
- 로그인 폼과 처리도 같은 방식으로 작성합니다.
# database adapter 설정하기
- Auth.js의 adapter는 사용자, 계정, 세션 등의 데이터를 저장하는데 사용하려는 데이터베이스 또는 백엔드 시스템에 애플리케이션을 연결합니다. 
- **자체 데이터베이스에 사용자 정보를 유지해야 하거나 특정 플로우를 구현하려는 경우가 아니라면 어댑터는 선택 사항입니다.** 예를 들어 이메일 제공업체는 인증 토큰을 저장할 수 있는 어댑터가 필요합니다.
- 공식 문서 > Getting started > Database Adapters > Official adapters에서 사용하는 서버리스 데이터베이스를 선택합니다.
- nextjs는 두 가지 세션 전략을 제공합니다:
	- 데이터베이스 세션 전략:  세션 데이터를 데이터베이스에 저장합니다. 데이터베이스 및 해당 adapter가 Edge 런타임/인프라와 호환되지 않으면 사용할 수 없습니다.(eg. Prisma)
	- JWT 세션 전략: JWT를 이용한 전략

> [!info] [[Prisma]] 기준으로 설명합니다.

1. `npm i @auth/prisma-adapter`를 실행합니다.
2. `prisma.schema`에 next auth에서 제공하는 기본적인 모델 스키마를 복붙합니다. 
> [!note] Session, Verification 모델을 사용하지 않습니다.
> - Verification 모델 대신 verification token을 직접 구현해서 credential provider로 회원 관련 기능을 만듭니다.
> - Prisma는 Edge 런타임에서 작동하지 않기 때문에 database session 전략을 사용할 수 없습니다.

> [!tip] User 모델의 password 필드를 선택 옵션으로 두는 이유는 OAuth 로그인 시 비밀번호를 필요로하지 않기 때문입니다.

```
model Account {
  id                 String  @id @default(cuid())
  userId             String
  type               String
  provider           String
  providerAccountId  String
  refresh_token      String?  @db.Text
  access_token       String?  @db.Text
  expires_at         Int?
  token_type         String?
  scope              String?
  id_token           String?  @db.Text
  session_state      String?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([provider, providerAccountId])
}
  
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  password      String?
  accounts      Account[]
}
```
# 비밀번호 암호화하기
1. bcrypt 라이브러리를 설치합니다.
```
npm i bcrypt
```

> [!warning] bcryptjs
> - bcrypt가 에러를 일으키는 경우가 종종있어 bcryptjs 라이브러를 대신 사용할 수 도 있습니다. 사용법은 거의 같습니다.
> ``` 
>npm i bcryptjs
>npm i -D @types/bcryptjs 
> ```

2. bcrypt는 type을 제공하지 않기때문에 type을 제공하는 라이러리를 설치합니다.
```
npm i -D @types/bcrypt
```

3. 비밀번호를 암호화합니다.
```ts
// actions/register.ts
import bcrypt from "bcrypt"
import { db } from "@/lib/db"

  const { email, password, name } = validateFields.data
  
  //암호화
  const hashedPassword = await bcrypt.hash(password, 10) 
  
  const existingUser = await db.user.findUnique({
    where: {
      email,
    },
  })
  
  if (existingUser) {
    return {
      error: `이미 회원으로 가입되어 있습니다. ${email}로 서비스를 이용해주세요.`,
    }
  }
  
  await db.user.create({
    data: {
      name,
      email,
      password: hashedPassword,
    },
  })
```
# next auth 설치 및 설정
1. next-auth v5 설치하기
```
yarn add next-auth@beta
```

2. 설정 파일 작성하기
```ts
// auth.ts
import NextAuth from "next-auth"
import GitHub from "next-auth/providers/github"
  
export const {
  handlers: { GET, POST },
  auth,
} = NextAuth({
  providers: [GitHub],
})
```
- `auth`: 인증 요청에 대한 처리를 담당하는 객체입니다. `middleware`에서 `auth`를 사용하여 사용자가 인증 요청 시 로그인 여부에 따라 이동할 수 있는 라우트를 지정합니다. 

3. API 라우트 작성하기
	- `app`라우트 안에 `api/auth/[...nextauth]/route.ts`를 생성합니다.
	- prisma는 edge를 지원하지 않기 때문에 `export const runtime = "edge"`를 삭제합니다.
```ts
// route.ts
export { GET, POST } from "@/auth"
```

4. 시크릿 키 작성하기
	- `.env`파일안에 시크릿키를 작성합니다.
```env
AUTH_SECRET="secret"
```

> [!warning] [missing secret 에러](https://authjs.dev/reference/core/errors#missingsecret)

5. OAuth에서 이용하는 api 확인하기
- `http://localhost:3000/api/auth/providers`로 접속하면 `auth.ts`에 설정한 provider에 대한 정보가 담긴 json형식의 데터가 화면에 나타납니다.
```json
{"github":{"id":"github","name":"GitHub","type":"oauth","signinUrl":"http://localhost:3000/api/auth/signin/github","callbackUrl":"http://localhost:3000/api/auth/callback/github"}}
```
# middleware 설정하기
- middleware는 next-auth의 구성 요소가 아니라 nextjs의 구성 요소입니다.

1. 프로젝트 파일 최상위에 `middleware.ts`를 만듭니다.
> [!warning] 파일이름을 정확히 `middleware`라고 해야 작동합니다.

2. 다음을 작성합니다. 호출할 미들웨어 경로를 [`matcher`](https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher)안에 적습니다. 이는 스프링시큐리티의 filterChain에서 인증 없이 접근 가능한 경로를 지정하는 것과 비슷합니다.
	- 특정 정적 파일과 이미지를 제외한 **모든 경로가 미들웨어를 호출**하도록 했습니다.
```ts
import { auth } from "./auth"
export default auth((req) => {
  // req.auth
})

export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
}
```
- `auth()` 안에서는 

> [!tip] `matcher`의 역할
> - `matcher`에 private나 public 경로 중 하나를 넣는게 아닙니다. `matcher`는 이를 체크하지 않습니다. **단지 미들웨어를 호출하기 위해 사용됩니다.**
> - `matcher`로 호출된 미들웨어는 위 `auth(... => ...)`함수를 호출합니다.

# Edge 호환 설정
- 일부 라이브러리 또는 ORM 패키지가 아직 표준 Web API를 반영하지 않은 경우 auth configuration을 여러 파일로 분할할 수 있습니다. 
- Prisma에서 Edge 호환을 설정하는 방법은 다음과 같습니다:

1. `auth.ts`와 같은 위치에 `auth.config.ts`를 만들고 다음을 적습니다.
```ts
import GitHub from "next-auth/providers/github"
import type { NextAuthConfig } from "next-auth"
  
export default {
  providers: [GitHub],
} satisfies NextAuthConfig
```

2. `auth.ts`를 다음과 같이 작성합니다.
```ts
import NextAuth from "next-auth"
import authConfig from "./auth.config"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { db } from "./lib/db"
  
export const {
  handlers: { GET, POST },
  auth,
} = NextAuth({
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
})
```

3. `middleware.ts`를 다음과 같이 작성합니다.
```ts
import NextAuth from "next-auth"
import authConfig from "@/auth.config"
  
const { auth } = NextAuth(authConfig)
  
export default auth((req) => {
  const isLoggedIn = !!req.auth
})
  
export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
}
```
# middleware로 라우트 지정하기
- 로그인 여부에 따라 접속할 수 있는 라우트를 정해둡니다.
```ts
//routes.ts
/**
 * 공개적으로 접근할 수 있는, 즉 로그아웃한 유저들도 접근 가능한 라우트들이 담긴 배열
 * 이 라우트들은 authentication을 필요로하지 않음
 * @type {string[]}
 * */
export const publicRoutes = ["/"]
  
/**
 * authentication에 사용되는 라우트들이 담긴 배열
 * 이 라우트들은 로그인한 유저를 /settings로 리다이렉트함
 * @type {string[]}
 * */
export const authRoutes = ["/auth/login", "/auth/register"]
  
/**
 * API authentication 라우트의 고정값(prefix)
 * 이 prefix로 시작하는 라우트는 API authentication을 위해 사용됨
 * @type {string}
 * */
export const apiAuthPrefix = "/api/auth"
  
/**
 * 로그인후 기본적으로 리다이렉트되는 경로
 * @type {string}
 * */
export const DEFAULT_LOGIN_REDIRECT = "/settings" //나중에 원하는 경로로 지정할 수 있음
```

- `middleware`에서 접속 가능한 URL을 지정합니다.
```ts
//middleware.ts
import NextAuth from "next-auth"
import authConfig from "@/auth.config"
import {
  apiAuthPrefix,
  DEFAULT_LOGIN_REDIRECT,
  authRoutes,
  publicRoutes,
} from "./routes"
  
const { auth } = NextAuth(authConfig)

export default auth((req) => {
  const { nextUrl } = req
  const isLoggedIn = !!req.auth

  const isApiAuthRoute = nextUrl.pathname.startsWith(apiAuthPrefix)
  const isPublicRoute = publicRoutes.includes(nextUrl.pathname)
  const isAuthRoute = authRoutes.includes(nextUrl.pathname)
  
  if (isApiAuthRoute) {
    return null
  }
  
  if (isAuthRoute) {
    if (isLoggedIn) {
      return Response.redirect(new URL(DEFAULT_LOGIN_REDIRECT, nextUrl))
    }
    return null
  }
  
  if (!isLoggedIn && !isPublicRoute) {
    return Response.redirect(new URL("/auth/login", nextUrl))
  }
  
  return null
})
  
export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
}
```
- `nextUrl`: 기본 URL입니다.(e.g. localhost:3000)
- `null`을 반환하는 것은 해당 라우트로 접속할 경우 아무 작업도 하지 않는 것을 의미합니다.

> [!tip] `isApiAuthRoute` 상수 할당
> 미들웨어의 작업이 "/api/auth/~"경로에 도달하는 것을 허용하기 위해 상수에 할당합니다. 
> next auth가 제대로 작동하려면 반드시 라우트를 auth함수 내부에 추가해야합니다.

> [!tip] `middleware`에서 `Response.redirect(new URL())`을 사용할때 
> `URL`생성자 안에 `nextUrl`을 추가해야 절대 경로를 만들어 지정한 경로로 이동할 수 있습니다.
# 로그인
- 모든 사용자가 서버액션으로 만들어둔 `login.ts`를 이용해 로그인하지는 않습니다. `/api/auth/`를 통해 로그인하는 사용자들도 있습니다. 
- 로그인을 시도하는 모든 사용자가 앱에서 요구하는 올바른 정보를 줬는지 체크하려면 `provider`에 `Credentials`를 추가하여 LoginSchema를 체크해야합니다.
```ts
// auth.config.ts
import type { NextAuthConfig } from "next-auth"
import Credentials from "next-auth/providers/credentials"
import { LoginSchema } from "@/schemas"
import { getUserByEmail } from "@/data/user"
import bcrypt from "bcryptjs"
  
export default {
  providers: [
    Credentials({
      async authorize(credentials, request) {
        const validateFields = LoginSchema.safeParse(credentials)
  
        if (validateFields.success) {
          const { email, password } = validateFields.data
  
          const user = await getUserByEmail(email)
          if (!user || !user.password) return null //회원이 아니거나 비밀번호가 없으면 작업 중지
  
          const passwordsMatch = await bcrypt.compare(password, user.password)
  
          if (passwordsMatch) return user
        }
  
        return null
      },
    }),
  ],
} satisfies NextAuthConfig
```
- 비밀번호가 없어도 작업을 중지하는 이유는, 외부 API를 이용해 로그인하는 사용자들은 비밀번호가 없기 때문에 일반적인 로그인 폼(비밀번호가 필요한)을 사용하여 로그인할 수 없도록 하기 위함입니다.

```ts
// auth.ts
import NextAuth from "next-auth"
import authConfig from "./auth.config"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { db } from "./lib/db"
  
export const {
  handlers: { GET, POST },
  //서버 컴포넌트나 서버 액션에서 사용 가능
  auth,
  signIn,
  signOut,
} = NextAuth({
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
})
```

```ts
// actions/login.ts
"use server"
  
import * as z from "zod"
import { LoginSchema } from "@/schemas"
import { signIn } from "@/auth" // auth.ts
import { DEFAULT_LOGIN_REDIRECT } from "@/routes"
import { AuthError } from "next-auth"
  
export const login = async (values: z.infer<typeof LoginSchema>) => {
  const validateFields = LoginSchema.safeParse(values)
  
  if (!validateFields.success) {
    return { error: "잘못된 입력입니다." }
  }
  
  const { email, password } = validateFields.data
  
  try {
    await signIn("credentials", {
      email,
      password,
      redirectTo: DEFAULT_LOGIN_REDIRECT,
    })
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case "CredentialsSignin":
          return { error: "회원이 아닙니다." }
        default:
          return { error: "잘못된 요청입니다." }
      }
    }
    throw error
  }
}
```
- 로그인 페이지에서 잘못된 이메일이나 비밀번호를 입력하면 `AuthError`가 발생하고, 이를 catch문에서 잡아 처리합니다.
- `AuthError`에 속하지 않는 에러가 발생하면 `throw`하고, `redirectTo`로 지정한 라우트로 리다이렉트되지 않습니다.
- 한 번 로그인한 후 다시 로그인 페이지나 회원가입 페이지로 가려하면 `middleware.ts`에서 정의한 경우(if문)에 따라 지정된 라우트로 자동 이동됩니다.  