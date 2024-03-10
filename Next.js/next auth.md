- next auth의 가장 중요한 목적은 OAuth입니다.
- 로그인 후 리다이렉션되는 페이지를 만들어야합니다.
- [초기 설정](https://next-auth.js.org/configuration/initialization)
- 리다이렉트 주소: `/api/auth/callback/:provider`
# [공식문서](https://authjs.dev/?_gl=1*1owi9s4*_gcl_au*Mjk1OTM2NDEwLjE3MTAwNDY5NDA.)

# 로그인, 회원가입 폼 만들기
## zod를 이용한 validation
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

```tsx
// components/auth/loginForm.tsx
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
import { LoginSchema } from "@/schemas"
import { useForm } from "react-hook-form"
import { Input } from "../ui/input"
import { Button } from "../ui/button"
import FormError from "../formError"
import FormSuccess from "../formSuccess"
import { login } from "@/actions/login"
import { useState, useTransition } from "react"
  
export function LoginForm() {
  const [error, setError] = useState<string | undefined>("")
  const [success, setSuccess] = useState<string | undefined>("")
  const [isPending, startTransition] = useTransition() //대기상태인지 여부 설정
  
  const form = useForm<z.infer<typeof LoginSchema>>({
    resolver: zodResolver(LoginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  })

  const onSubmit = (values: z.infer<typeof LoginSchema>) => {
    setError("")
    setSuccess("")
  
    startTransition(() => {
      login(values).then((data) => {
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

2. bcrypt는 type을 제공하지 않기때문에 type을 제공하는 라이러리를 설치합니다.
```
npm i -D @types/bcrypt
```

3. 비밀번호를 암호화합니다.
```tsx
//...
const hashedPassword = await bcrypt.hash(password, 10)
```