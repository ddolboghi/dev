`jwt()` 콜백이 호출될때 필드를 저장하려고 다음과 같이 코드를 작성했더니 `JWTSessionError`가 발생했습니다. 
```ts
// auth.js
import NextAuth from "next-auth"
import authConfig from "./auth.config"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { db } from "./lib/db"
  
export const {
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  callbacks: {
    async session({ token, session }) {
      if (sessionStorage.user) {
        session.user.customField = token.customField
      }
      return session
    },
    async jwt({ token, user, account, profile, session, trigger }) {
      token.customField = "1q2w3e4r"
      return token
    },
  },
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
})
```

```
[auth][error] JWTSessionError: Read more at https://errors.authjs.dev#jwtsessionerror
[auth][cause]: ReferenceError: sessionStorage is not defined
    at Object.session (webpack-internal:///(rsc)/./auth.ts:20:13)
    at Object.session (webpack-internal:///(rsc)/./node_modules/next-auth/lib/index.js:31:51)
    at Module.session (webpack-internal:///(rsc)/./node_modules/@auth/core/lib/actions/session.js:41:52)
    at async AuthInternal (webpack-internal:///(rsc)/./node_modules/@auth/core/lib/index.js:47:24)
    at async Auth (webpack-internal:///(rsc)/./node_modules/@auth/core/index.js:126:34)
    at async SettingsPage (webpack-internal:///(rsc)/./app/(protected)/settings/page.tsx:20:21)
[auth][details]: {}
```

에러 로그를 잘보면 sessionStorage가 정의되지 않아 참조할 수 없다고 합니다.  

코드를 다시 보니 session.user.customField에 값을 할당하기 전에 if 문에서  `sessionStorage.user`를 참조하고 있었습니다. 애초에 sessionStorage를 정의한 적이 없으므로 원래 코드인 `session.user`로 수정했더니 정상작동했습니다.

