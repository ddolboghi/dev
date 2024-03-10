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

> [!info] [[Prisma]] 기준으로 설명합니다.

1. `npm i @auth/prisma-adapter`를 실행합니다.
2. `prisma.schema`에 next auth에서 제공하는 기본적인 모델 스키마를 복붙합니다. 
> [!info] 여기서는 Session, Verification 모델을 사용하지 않습니다.
> - Verification 모델 대신 verification token을 직접 구현해서 credential provider로 회원 관련 기능을 만듭니다.

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

3. API 라우트 작성하기
	- `app`라우트 안에 `api/auth/[...nextauth]/route.ts`를 생성합니다.
	- prisma는 edge를 지원하지 않기 때문에 `export const runtime = "edge"`를 삭제합니다.
```ts
// route.ts
export { GET, POST } from "@/auth"
```
