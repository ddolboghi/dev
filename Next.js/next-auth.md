- next auth의 가장 중요한 목적은 OAuth입니다.
- 로그인 후 리다이렉션되는 페이지를 만들어야합니다.
- [초기 설정](https://next-auth.js.org/configuration/initialization)
- OAuth 리다이렉트 주소: `/api/auth/callback/:provider`

> [!warning] 아래 글들은 next-auth **v5** @beta 기준입니다.

# [공식문서](https://authjs.dev/guides/upgrade-to-v5)

[애플로 로그인](https://developer.apple.com/kr/sign-in-with-apple/get-started/) -> apple developer program 가입해야함 (129,000원/1년)

# 설치

```
yarn add next-auth@beta
```

# 로그인, 회원가입 폼 만들기

## zod를 이용한 validation

- `index.ts`: `zod`를 이용해 검증 schema를 작성합니다.

```ts
// lib/index.ts
import * as z from "zod";

export const LoginSchema = z.object({
  email: z.string().email({
    message: "이메일은 필수 입력 항목입니다.",
  }),
  password: z.string().min(1, {
    message: "비밀번호는 필수 입력 항목입니다.",
  }),
});

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
});
```

- `react-hook-form`라이브러리를 사용하지 않는다면 `useFormState`를 사용해서 `<form>`의 `action`속성에 formAction을 넘겨줍니다.

- `register.ts`: 사용자가 입력한 값을 받아 처리하는 곳입니다.

```ts
// actions/register.ts
"use server";

import * as z from "zod";
import { RegisterSchema } from "@/schemas";

export const register = async (values: z.infer<typeof RegisterSchema>) => {
  const validateFields = RegisterSchema.safeParse(values);

  if (!validateFields.success) {
    return { error: "Invalid fields!" };
  }

  return { success: "Email sent!" };
};
```

- 로그인 폼과 처리도 같은 방식으로 작성합니다.

# database adapter 설정하기

- Auth.js의 adapter는 사용자, 계정, 세션 등의 데이터를 저장하는데 사용하려는 데이터베이스 또는 백엔드 시스템에 애플리케이션을 연결합니다.
- **자체 데이터베이스에 사용자 정보를 유지해야 하거나 특정 플로우를 구현하려는 경우가 아니라면 어댑터는 선택 사항입니다.** 예를 들어 이메일 제공업체는 인증 토큰을 저장할 수 있는 어댑터가 필요합니다.
- 공식 문서 > Getting started > Database Adapters > Official adapters에서 사용하는 서버리스 데이터베이스를 선택합니다.
- nextjs는 두 가지 세션 전략을 제공합니다:
  - 데이터베이스 세션 전략: 세션 데이터를 데이터베이스에 저장합니다. 데이터베이스 및 해당 adapter가 Edge 런타임/인프라와 호환되지 않으면 사용할 수 없습니다.(eg. Prisma)
  - JWT 세션 전략: JWT를 이용한 전략

> [!info] [[Prisma]] 기준으로 설명합니다.

1. `npm i @auth/prisma-adapter`를 실행합니다.
2. `prisma.schema`에 next auth에서 제공하는 기본적인 모델 스키마를 복붙합니다.
   > [!note] Session, Verification 모델을 사용하지 않습니다.
   >
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
>
> - bcrypt가 에러를 일으키는 경우가 종종있어 bcryptjs 라이브러를 대신 사용할 수 도 있습니다. 사용법은 거의 같습니다.
>
> ```
> npm i bcryptjs
> npm i -D @types/bcryptjs
> ```

2. bcrypt는 type을 제공하지 않기때문에 type을 제공하는 라이러리를 설치합니다.

```
npm i -D @types/bcrypt
```

3. 비밀번호를 암호화합니다.

````ts
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
    data의 인증 여부에 따라 이동할 수 있는 라우트를 지정합니다.

3. API 라우트 작성하기
	- `app`라우트 안에 `api/auth/[...nextauth]/route.ts`를 생성합니다.

> [!info] `route.ts`는 next-auth외 다른 라이브러리가 사용할 수 있도록 여러 라우 핸들러를 스캐폴딩할 수 있는 포괄적인 라우트입니다.

> [!tip] prisma는 edge를 지원하지 않기 때문에 `export const runtime = "edge"`를 삭제합니다.

```ts
// route.ts
export { GET, POST } from "@/auth"
````

4. 시크릿 키 작성하기
   - `.env`파일안에 시크릿키를 작성합니다.

```env
AUTH_SECRET="secret"
```

> [!warning] [missing secret 에러](https://authjs.dev/reference/core/errors#missingsecret)

5. OAuth에서 이용하는 api 확인하기

- `http://localhost:3000/api/auth/providers`로 접속하면 `auth.ts`에 설정한 provider에 대한 정보가 담긴 json형식의 데터가 화면에 나타납니다.

```json
{
  "github": {
    "id": "github",
    "name": "GitHub",
    "type": "oauth",
    "signinUrl": "http://localhost:3000/api/auth/signin/github",
    "callbackUrl": "http://localhost:3000/api/auth/callback/github"
  }
}
```

# middleware 설정하기

- middleware는 next-auth의 구성 요소가 아니라 nextjs의 구성 요소입니다.

1. 프로젝트 파일 최상위에 `middleware.ts`를 만듭니다.

   > [!warning] 파일이름을 정확히 `middleware`라고 해야 작동합니다.

2. 다음을 작성합니다. 호출할 미들웨어 경로를 [`matcher`](https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher)안에 적습니다. 이는 스프링시큐리티의 filterChain에서 인증 없이 접근 가능한 경로를 지정하는 것과 비슷합니다.
   - 특정 정적 파일과 이미지를 제외한 **모든 경로가 미들웨어를 호출**하도록 했습니다.

```ts
// middleware.ts
import { auth } from "./auth";
export default auth((req) => {
  // req.auth
});

export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

- `auth()` 안에서는 인증 요청 시 사용자의 인증 완료 여부에따라 접속가능한 라우트를 정의합니다.

> [!tip] `matcher`의 역할
>
> - `matcher`에 private나 public 경로 중 하나를 넣는게 아닙니다. `matcher`는 이를 체크하지 않습니다. **단지 미들웨어를 호출하기 위해 사용됩니다.**
> - `matcher`로 호출된 미들웨어는 위 `auth(... => ...)`함수를 호출합니다.

## Edge 호환 설정

- 일부 라이브러리 또는 ORM 패키지가 아직 표준 Web API를 반영하지 않은 경우 auth configuration을 여러 파일로 분할할 수 있습니다.
- Prisma에서 Edge 호환을 설정하는 방법은 다음과 같습니다:

1. `auth.ts`와 같은 위치에 `auth.config.ts`를 만들고 다음을 적습니다.

```ts
// auth.config.ts
import GitHub from "next-auth/providers/github";
import type { NextAuthConfig } from "next-auth";

export default {
  providers: [GitHub],
} satisfies NextAuthConfig;
```

2. `auth.ts`를 다음과 같이 작성합니다.

```ts
// auth.ts
import NextAuth from "next-auth";
import authConfig from "./auth.config";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { db } from "./lib/db";

export const {
  handlers: { GET, POST },
  auth,
} = NextAuth({
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
});
```

- `NextAuth`를 spread하여 얻을 수 있는 객체에는 `auth`, `signIn`, `signOut`등이 있습니다. 이들은 **서버 컴포넌트**나 **서버 액션**에서만 사용할 수 있습니다.

> [!tip] 로그인/로그아웃을 완전히 클라이언트 컴포넌트에서만 할 수도 있습니다.

3. `middleware.ts`를 다음과 같이 작성합니다.

```ts
// middleware.ts
import NextAuth from "next-auth";
import authConfig from "@/auth.config";

const { auth } = NextAuth(authConfig);

export default auth((req) => {
  const isLoggedIn = !!req.auth;
});

export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

- **`auth.config.ts`가 기본 제공되는 `auth`의 내부 동작을 대체합니다.**

# middleware로 라우트 지정하기

- `middleware`에는 수많은 경우에 대해 리다이렉트되는 라우트를 지정하거나 다른 작업을 수행할 수 있습니다.
- 아래는 로그인 여부에 따라 접속할 수 있는 라우트들입니다.

```ts
//routes.ts
/**
 * 공개적으로 접근할 수 있는, 즉 로그아웃한 유저들도 접근 가능한 라우트들이 담긴 배열
 * 이 라우트들은 authentication을 필요로하지 않음
 * @type {string[]}
 * */
export const publicRoutes = ["/"];

/**
 * authentication에 사용되는 라우트들이 담긴 배열
 * 이 라우트들은 로그인한 유저를 /settings로 리다이렉트함
 * @type {string[]}
 * */
export const authRoutes = ["/auth/login", "/auth/register"];

/**
 * API authentication 라우트의 고정값(prefix)
 * 이 prefix로 시작하는 라우트는 API authentication을 위해 사용됨
 * @type {string}
 * */
export const apiAuthPrefix = "/api/auth";

/**
 * 로그인후 기본적으로 리다이렉트되는 경로
 * @type {string}
 * */
export const DEFAULT_LOGIN_REDIRECT = "/settings"; //나중에 원하는 경로로 지정할 수 있음
```

- `middleware`에서 접속 가능한 URL을 지정합니다.

```ts
//middleware.ts
import NextAuth from "next-auth";
import authConfig from "@/auth.config";
import {
  apiAuthPrefix,
  DEFAULT_LOGIN_REDIRECT,
  authRoutes,
  publicRoutes,
} from "./routes";

const { auth } = NextAuth(authConfig);

export default auth((req) => {
  const { nextUrl } = req;
  const isLoggedIn = !!req.auth;

  const isApiAuthRoute = nextUrl.pathname.startsWith(apiAuthPrefix);
  const isPublicRoute = publicRoutes.includes(nextUrl.pathname);
  const isAuthRoute = authRoutes.includes(nextUrl.pathname);

  if (isApiAuthRoute) {
    return null;
  }

  if (isAuthRoute) {
    if (isLoggedIn) {
      return Response.redirect(new URL(DEFAULT_LOGIN_REDIRECT, nextUrl));
    }
    return null;
  }

  if (!isLoggedIn && !isPublicRoute) {
    return Response.redirect(new URL("/auth/login", nextUrl));
  }

  return null;
});

export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
};
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
- 로그인을 시도하는 모든 사용자가 앱에서 요구하는 올바른 정보를 줬는지 체크하려면 `providers`에 `next-auth/providers/credentials`를 추가하여 LoginSchema를 체크해야합니다.
  이때 `authorize()`에 넘겨지는 `credentials` object에는 로인 시 사용자가 입력한 값들과 `callbackUrl`이 들어있습니다.

```
{
  email: 'test@example.com',
  password: '1234567',
  callbackUrl: '/settings'
}
```

```ts
// auth.config.ts
import type { NextAuthConfig } from "next-auth";
import Credentials from "next-auth/providers/credentials";
import { LoginSchema } from "@/schemas";
import { getUserByEmail } from "@/data/user";
import bcrypt from "bcryptjs";

export default {
  providers: [
    Credentials({
      async authorize(credentials, request) {
        const validateFields = LoginSchema.safeParse(credentials);

        if (validateFields.success) {
          const { email, password } = validateFields.data;

          const user = await getUserByEmail(email);
          if (!user || !user.password) return null; //회원이 아니거나 비밀번호가 없으면 작업 중지

          const passwordsMatch = await bcrypt.compare(password, user.password);

          if (passwordsMatch) return user;
        }

        return null;
      },
    }),
  ],
} satisfies NextAuthConfig;
```

- 비밀번호가 없어도 작업을 중지하는 이유는, 외부 API를 이용해 로그인하는 사용자들은 비밀번호가 없기 때문에 일반적인 로그인 폼(비밀번호가 필요한)을 사용하여 로그인할 수 없도록 하기 위함입니다.

```ts
// auth.ts
import NextAuth from "next-auth";
import authConfig from "./auth.config";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { db } from "./lib/db";

export const {
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
});
```

```ts
// actions/login.ts
"use server";

import * as z from "zod";
import { LoginSchema } from "@/schemas";
import { signIn } from "@/auth"; // auth.ts
import { DEFAULT_LOGIN_REDIRECT } from "@/routes";
import { AuthError } from "next-auth";

export const login = async (values: z.infer<typeof LoginSchema>) => {
  const validateFields = LoginSchema.safeParse(values);

  if (!validateFields.success) {
    return { error: "잘못된 입력입니다." };
  }

  const { email, password } = validateFields.data;

  try {
    await signIn("credentials", {
      email,
      password,
      redirectTo: DEFAULT_LOGIN_REDIRECT,
    });
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case "CredentialsSignin":
          return { error: "회원이 아닙니다." };
        default:
          return { error: "잘못된 요청입니다." };
      }
    }
    throw error;
  }
};
```

- 로그인 페이지에서 잘못된 이메일이나 비밀번호를 입력하면 `AuthError`가 발생하고, 이를 catch문에서 잡아 처리합니다.
- `AuthError`에 속하지 않는 에러가 발생하면 `throw`하고, `redirectTo`로 지정한 라우트로 리다이렉트되지 않습니다.
- 한 번 로그인한 후 다시 로그인 페이지나 회원가입 페이지로 가려하면 `middleware.ts`에서 정의한 경우(if문)에 따라 지정된 라우트로 리다이렉트됩니다.

> [!tip] `actions/login.ts`에서 수행되는 작업들은 `callbacks`에 똑같이 작성할 수 있습니다.

# callbacks

- `callbacks`는 `authorize` 에서 로그인 로직을 수행하고 나서 마지막으로 실행되는 부분입니다.
- `callbacks`는 데이터베이스 없이 액세스 제어를 구현할수 있으며, 외부 데이터베이스나 API와 통합할 수도 있어 JWT를 사용할때 매우 강력합니다.

> [!tip]
> JWT를 사용할 때 액세스 토큰이나 사용자 ID와 같은 데이터를 브라우저에 전달하려면 `jwt` 콜백이 호출될 때 토큰의 데이터를 유지한 다음 `session` 콜백에서 브라우저에 데이터를 전달할 수 있습니다.

- `auth.ts`의 `NextAuth`객체에 `callbacks`객체를 넘겨줄 수 있습니다.

```ts
// auth.ts
export const {
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  callbacks: {
    //핸들러 지정
  },
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
});
```

## `signIn()` callback

> [!warning] signIn() 출처: `import signIn from “@/auth”` → server side에서만 사용 가능! (auth()의 프로퍼티이기 때문)

`import signIn from “next-auth/react”` → client side에서만 사용 가능!

- `signIn()` 콜백을 사용하여 사용자의 로그인 허용 여부를 제어합니다.
- `true`를 반환하면 로그인되고, `false`나 URL을 반환하면 로그인되지 않습니다.

```ts
callbacks: {
  async signIn({ user, account, profile, email, credentials }) {
    const isAllowedToSignIn = true
    if (isAllowedToSignIn) {
      return true
    } else {
      //에러 메시지를 보여주기 위해 false 반환
      return false
      //또는 리다이렉트할 URL 반환
      // return '/unauthorized'
    }
  }
}
```

- 위의 `signIn()`콜백에 의해 반환된 리다이렉트는 인증을 취소하기 때문에 인증 성공 시(로그인 성공 시)에는 URL이 반환되면 안되고 반드시 true를 반환해야합니다.

### CredentialsProvider 사용 시

- CredentialsProvider 사용 시 `signIn`콜백의 `user`객체는 credential provider의 `authorize` 콜백에서 반환된 응답이고, `profile`객체는 HTTP POST 제출의 body입니다.
- **`authorize`후 로그인 되기 전에 추가 확인 후 로그인을 허용할지 말지 결정합니다.**
- `jwt`콜백에서 token과 user를 같은 위치로 만들고 리턴 합니다. 그러면 `jwt`가 다시 `token`을 리턴 하게 되는데 `session`콜백에서 `session.user`에 그 값을 지정하고 다시 `session`을 리턴 합니다.
  ```tsx
  async session({ session, token }) {
    session.user = token as any
    return session
  },
  async jwt({ token, user }) {
    return { ...token, ...user }
  },
  ```

> [!info] 데이터베이스 사용 유무에 따른 `user`객체의 형태
> 데이터베이스와 Auth.js를 함께 사용하는 경우
>
> - 사용자가 이전에 로그인한 적이 있는 경우 데이터베이스의 `user`객체(사용자 ID 포함)가 됩니다.
> - 이전에 로그인한 적이 없는 경우 간단한 prototype `user`객체(e.g. 이름, 이메일, 이미지)가 됩니다.
>
> 데이터베이스 없이 Auth.js를 사용하는 경우
>
> - `user` 객체는 항상 프로필에서 추출된 정보가 포함된 prototype `user`객체가 됩니다.

## `redirect()` callback

- redirect 콜백은 사용자가 콜백 URL로 리다이렉트될때마다 호출됩니다.
- 사이트 URL과 같은 URL만 허용됩니다.
- redirect 콜백은 동일한 흐름에서 여러번 호출될 수 있습니다.
- `middleware.ts`에서 경우에따른 리다이렉트를 잘 지정했다면 redirect 콜백을 많이 사용할 일이 없습니다.

```ts
callbacks: {
  async redirect({ url, baseUrl }) {
    // Allows relative callback URLs
    if (url.startsWith("/")) return `${baseUrl}${url}`
    // Allows callback URLs on the same origin
    else if (new URL(url).origin === baseUrl) return url
    return baseUrl
  }
}
```

## `session()` callback
- session 콜백은 session이 확인될때마다 호출됩니다.
- `session()` 콜백은 반드시 `session`객체를 반환해야 합니다.
- 보안을 위해 기본적으로 토큰의 하위 집합만 반환됩니다.
- `jwt()` 콜백을 통해 토큰에 추가한 내용을 사용하려면 session 콜백에 명시적으로 전달해야 클라이언트(e.g. `getSession()`, `useSession()`, `/api/auth/session`)가 사용할 수 있습니다.
- 데이터베이스 세션을 사용할때는 `User` 객체가 인자로 전달됩니다.
- 세션에 JWT를 사용할때는 JWT payload가 전달됩니다.
****
```ts
callbacks: {
  async session({ session, token, user }) {
    // Send properties to the client, like an access_token from a provider.
    session.accessToken = token.accessToken
    return session
  }
}
```

> [!tip]
> JWT를 사용할때 **`jwt()`콜백은 `session()`콜백 전에 호출**되므로, JWT에 추가한 어떤 것이든 session 콜백에서 즉시 사용할 수 있습니다.

> [!warning]
>
> - 데이터베이스 세션을 사용할때도 session 객체는 서버에 저장되지 않습니다. 세션 토큰, 사용자, 만료 시간 등이 세션 테이블에 저장됩니다.
> - 세션 데이터를 서버 측에서 유지해야 하는 경우 세션에 대해 반환된 accessToken을 키로 사용하고 `session()` 콜백에서 데이터베이스에 연결하여 액세스할 수 있습니다. 세션 accessToken 값은 rotate하지 않으며 세션이 유효한 한 유효합니다???
> - 데이터베이스 세션 대신 JWT를 사용한다면, 토큰에 저장된 User ID나 unique key를 사용해야합니다.
> - JWT를 사용할 때는 세션 용 액세스 토큰이 생성되지 않으므로 로그인 시 직접 키를 생성해야 합니다.
### jwt 콜백에서 추가한 accessToken을 세션에 저장하기
* `session.user`에 jwt콜백에서 넘겨주는 `token`을 할당
* `session`을 리턴하면 클라이언트 컴포넌트에서 `{data:session} = useSession()`으로 `session.user.accessToken` 접근 가능
```ts
async session({ session, token }) {
	session.user = token
	return session
},
```

## `jwt()` callback
- JWT가 만들어질때마다 jwt 콜백이 호출됩니다.(e.g. 로그인 시, 클라이언트에서 세션 접근 시)
- `jwt()` 콜백은 반드시 `token`을 반환합니다. 반환값은 암호화되있으며, 쿠키에 `session-token`으로 저장됩니다.
- 로그인 시 `token`은 다음과 같이 구성됩니다:
  ```
  {
    token: {
      name: '시월이',
      email: 'siwoli@example.com',
      picture: null,
      sub: 'cltl4vlvp00006t5j7ub0a113',
      iat: 1710140570,
      exp: 1712732570,
      jti: '660d7b86-4693-4865-8041-9418e1e9fa16'
    }
  }
  ```
- `jwt()`의 파라미터:

  - `token`: `session({ token })`의 `token`과 완전히 동일합니다.
  - `sub`: **데이터베이스 세션**에 저장된 데이터의 id 값입니다. OAuth provider가 제공하는 고유 id입니다.
  - `user` , `profile`, `isNewUser`는 provider와 데이터베이스 사용 여부에 따라 달라집니다.
  - `user`, `account`, `profile`, `isNewUser`는 jwt 콜백이 처음 호출될 때(로그인 중일때)만 전달됩니다. **이후 호출에서는 `token`만 사용할 수 있습니다.**

- JWT 세션을 사용하고 있다면, `/api/auth/signin`, `/api/auth/session`로 요청하거나 `getSession()`, `getServerSession()`, `useSession()`를 호출할때 `jwt()` 콜백을 호출합니다. 데이터베이스 세션 사용 시 호출되지 않습니다.
- **토큰 만료 시간은 세션이 활성화될때마다 연장됩니다.**
- 이 토큰에 User ID, OAuth 액세스 토큰과 같은 데이터를 유지할 수 있습니다. 브라우저에서 사용할 수 있도록 하려면 `session()` 콜백도 확인해야합니다.
### 백엔드에서 받은 accessToken 추가하기
- `authorize`에서 return한 `user`에 들어있는 accessToken을 session에 저장하기 위해 동일 선상에 추가해서 return합니다.
```ts
async jwt({ token, user }) {
  return { ...token, ...user }
},
```

## token에 추가한 데이터의 타입 지정
- token에 새로 추가된 데이터의 타입은 직접 지정해줘야 합니다.
- User 모델에 `enum`타입인 `role`을 새로 만들었고, token에 `role`을 추가했을때 다음과 같이 타입을 지정했습니다.

```prisma
// schema.prisma
enum UserRole {
  ADMIN
  USER
}
```

```ts
// auth.ts
declare module "next-auth" {
  interface User {
    role: string;
  }
}

declare module "next-auth/jwt" {
  interface JWT {
    role?: string;
  }
}
```

# OAuth

- Github, Google OAuth를 사용합니다.

1. `auth.config.ts`의 providers에 Google, Github provider를 추가해줍니다.

```ts
import GitHub from "next-auth/providers/github"
import Google from "next-auth/providers/google"
//...
providers: [
	Google({
	  clientId: process.env.GOOGLE_CLIENT_ID,
	  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
	}),
	GitHub({
	  clientId: process.env.GITHUB_CLIENT_ID,
	  clientSecret: process.env.GITHUB_CLIENT_SECRET,
	}),
	//...
```

2. `.env`에 발급받은 CLIENT_ID, CLIENT_SECRET를 작성합니다.

3. `signIn()`에 provider의 id값을 넘겨주고, `callbackUrl`을 지정해줍니다. 이 방식은 서버 컴포넌트에서 진행했던 `Credential` 작업과 다르게 클라이언트 컴포넌트에서 할 수 있습니다.

```tsx
"use client";
import { signIn } from "next-auth/react";
//...
function Social() {
  const onClick = (provider: "google" | "github") => {
    signIn(provider, {
      callbackUrl: DEFAULT_LOGIN_REDIRECT,
    });
  };

  return (
    <div className="flex items-center w-full gap-x-2">
           {" "}
      <Button
        size="lg"
        className="w-full"
        variant="outline"
        onClick={() => onClick("google")}
      >
                <FcGoogle className="h-5 w-5" />     {" "}
      </Button>
            <Button
        size="lg"
        className="w-full"
        variant="outline"
        onClick={() => onClick("github")}
      >
                <FaGithub className="h-5 w-5" />     {" "}
      </Button>   {" "}
    </div>
  );
}
```

## OAuthAccountNotLinked: Another account already exists with the same e-mail address.

- **로그인하려는 유저의 이메일**과 **동일한 이메일이 DB에 존재할때** next-auth가 보안 상 이미 존재하는 계정을 새로 로그인하는 계정과 동기화하지 않으려고 하기 때문에 발생하는 오류입니다.
- 하나의 계정으로 자동 동기화하려면, provider에 `allowDangerousEmailAccountLinking: true`를 추가하면 됩니다.

```tsx
providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      allowDangerousEmailAccountLinking: true,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    }),
```

- 자동 동기화는 신뢰할만한 provider가 아닌 이상 보안에 좋지 않고, 유저의 선택 없이 자동으로 동기화합니다.
- 계정 자동 동기화보다는 로그인을 막고, 계정 동기화 권한을 유저에게 주는게 바람직합니다.

# Events

- 이벤트는 응답을 반환하지 않는 비동기 함수이며, 로그/리포팅 또는 사이드이펙트를 처리하는 데 유용합니다.
- 디버깅 또는 로그를 위해 아래 이벤트 중 하나에 대한 핸들러를 지정할 수 있습니다.
  > [!NOTE]
  >
  > - 인증 API의 실행은 이벤트 핸들러의 `await`에 의해 차단됩니다.
  > - 이벤트 핸들러가 부담스러운 작업을 시작하는 경우, 해당 작업에 대한 promise를 차단해서는 안 됩니다.

# 세션에 접근하는 방법(v4.xx)
## getToken()
- NextAuth.js 요청(`req`)을 받아 NextAuth.js에서 발급한 JWT의 페이로드 또는 원시 JWT 문자열을 반환합니다. 
- **쿠키** 또는 **`Authorization` 헤더**에서 JWT를 찾습니다.
- JWT를 사용하는 경우 `getToken()` 반환값인 속성(`emial`, `name`, `picture`, `sub`)에 접근할 수 있습니다. 이는 JWT를 해독한 값이며, `DefaultJWT`타입에 정의된 값들입니다.  
  이 방법은 **서버 사이드에서만 사용**할 수 있습니다.
- `NEXTAUTH_URL`환경 변수를 설정하고 애플리케이션이 JWT 쿠키를 읽을 수 있는 경우(e.g. 동일한 도메인) 모든 애플리케이션에서 `getToken()`을 사용할 수 있습니다.
- `getToken()`에 `api/auth/[...nextauth]`에서 사용하는 시크릿키를 파라미터로 넘겨줘야 합니다.
```js
// This is an example of how to read a JSON Web Token from an API route
import jwt from "next-auth/jwt"

const secret = process.env.SECRET

export default async (req, res) => {
  const token = await jwt.getToken({ req, secret })
  if (token) {
    // Signed in
    console.log("JSON Web Token", JSON.stringify(token, null, 2))
  } else {
    // Not Signed in
    res.status(401)
  }
  res.end()
}
```
## getSession()
- **세션**에 저장된 정보를 가져옵니다.
- `NextAuth`의 `session`콜백은 
- `NextAuth`에서 `callbacks` 흐름에 따라 세션에 저장된 정보들에 접근할 수 있습니다.
```js
import { getSession } from "next-auth/client"

export default async (req, res) => {
  const session = await getSession({ req })
  if (session) {
    // Signed in
    console.log("Session", JSON.stringify(session, null, 2))
  } else {
    // Not Signed in
    res.status(401)
  }
  res.end()
}
```
