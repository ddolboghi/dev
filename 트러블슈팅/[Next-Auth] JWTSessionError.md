next-auth 공식문서의 팁대로 `jwt()` 콜백이 처음 호출될때 필드를 저장하려고 다음과 같이 코드를 작성했습니다. 
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
      if (user || account || profile || session || trigger) {
        token.customField = "1q2w3e4r"
      }
      return token
    },
  },
  adapter: PrismaAdapter(db),
  session: { strategy: "jwt" },
  ...authConfig,
})
```

callbacks의 jwt 콜백을 지정하는 부분에서 JWTSessionError가 발생했습니다. 
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

[에러 로그에 있는 사이트](https://authjs.dev/reference/core/errors/#jwtsessionerror)를 들어가보니
> Auth.js가 JWT 기반 세션(`strategy: "jwt"`)을 디코딩하거나 인코딩할 수 없는 경우 발생합니다.