# Next.js 란?
- 풀스택 웹 애플리케이션을 구축하기 위한 리액트 프레임워크입니다.
# Next.js의 서버 사이드 렌더링(SSR)과 정적사이트 생성(SSG)
- 서버 사이드 렌더링: 각 요청에 대해 서버에서 html페이지를 생성해 초기 렌더링에 필요한 모든 데이터와 마크업 포함 --> SEO 개선, 퍼포먼스 개선, 사용자 경험 향상, 소셜 미디어 공유 개선
# Next.js의 주요 기능 및 장점
- 자동 코드 분할: 각 페이지에서 사용되는 컴포넌트를 기반으로 애플리케이션 코드를 자동으로 작은 번들로 분할해 사용자가 현재 페이지에 필요한 코드만 다운로드되도록 합니다. 이를 통해  로딩 시간이 단축되고 성능이 향상됩니다.
- 핫 모듈 교체(HMR): 개발 시 전체 페이지를 새로고침하지 않아도 애플리케이션의 변경 사항을 확인할 수 있습니다.
- 자동 페이지 라우팅: `app` 디렉토리의 파일 구조를 기반으로 자동으로 페이지 라우팅을 처리합니다.
> [!페이지 라우팅]
> - 웹 애플리케이션에서 사용자가 요청한 URL 경로에 따라 적절한 페이지를 표시하는 기술 또는 프로세스입니다.
> - 페이지 라우팅을 통해 SPA(Single-Page Application)는 사용자가 애플리케이션 내에서 여러 페이지를 탐색하고 상호작용할 수 있도록 합니다.
- 통합된 API 개발: 개발자들이 애플리케이션 내에서 쉽게 서버리스 API 경로를 생성할 수 있도록 지원해, 풀스택 애플리케이션 개발 프로세스를 간소화하고 프론트엔드와 백엔드 간 원할하게 통신할 수 있습니다.
- 하이브리드 렌더링: 서버 사이드 렌더링과 정적 사이트 생성 모두 지원해 특정 방식을 선택하거나 하나의 애플리케이션 내 두 기술을 결합할 수 있습니다.
- 내장된 CSS 및 Javascript 지원: 인기있는 CSS-in-JS를 포함해, ES모듈과 dynamic import 같은 현대적인 Javascript 기능을 지원합니다.
- 데이터 가져오기: 서버 컴포넌트에서 async/await를 사용해 간소화된 데이터 가져오기, 요청 메모이제이션, 데이터 캐싱 및 재검증을 위한 확장된 fetch API를 제공합니다.
- 확장성: 플러그인, 미들웨어, 사용자 정의 설정을 통해 쉽게 확장할 수 있습니다.

---
# 설치 방법
1. 로컬 전역에 next.js를 설치합니다.
```
npm install -g create-next-app
```

2. next.js 앱을 만듭니다.
```
npx create-next-app@latest
```
- 앱 이름 작성
- 타입스크립트 사용 yes
- src 디렉토리 사용 no (대부분의 next.js 프로젝트들은 source 디렉토리를 사용하지 않습니다.)
- App Router 사용 yes
- import 축약어 커스터마이징 no

 개발 버전의 앱을 실행하는 명령어는 다음과 같습니다.
```
npm run dev
```
# 기본 파일 및 디렉토리
- app: app router라고도하며, 라우팅 시스템을 위한 컨테이너 
- page.tsx: 메인페이지
- layout.tsx: 메인페이지 감싸는 용도의 페이지
- public: static 파일 보관용 디렉토리
- next.config.js: next.js 설정 파일
- node_modules: 설치한 라이브러리 보관용 폴더
- .eslintrc.json: 정적 코드 분석 툴인 ESLint 설정에 관한 파일

> [!warning] globals.css는 앱 전체 스타일 설정만 해야합니다.
> globals.css에 특정 컴포넌트나 페이지에 대한 css를 작성하면 관리하기 어려워집니다. 따라서 앱 전체에 적용되는 css만 작성하는게 좋습니다.
# prettier 설정
- prettier는 코드 포맷터로, 코드 스타일을 지정해 일관성있게 관리해줍니다.
1. prettier 설치
```
npm install --save-dev prettier
```
2. prettier 설정
- 프로젝트 폴더에 `.prettierrc`파일 생성하고 다음과 같이 작성합니다.
```
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all"
}
```
- semi: 코드 끝에 세미콜론을 붙일지(**nextjs에서는 코드 끝에 세미콜론을 붙이지 않습니다.**) 
- singleQuote: 홑따옴표를 사용할지
- tabWidth: 들여쓰기 칸 수
- trailingComma: 쉼표를 어디까지 붙일지
	- all: 객체나 배열을 작성 할 때, 원소 혹은 key-value 의 맨 뒤에있는 것에도 쉼표를 붙입니다.
- printWidth: 한 줄 최대 칸 수
- [기타 옵션](https://prettier.io/docs/en/options.html)
# 포트 번호 변경 방법
package.json의 `next`명령어들 뒤에 `-p {포트번호}`를 작성합니다.
```json
"scripts": {
    "dev": "next dev -p 3001",
    "build": "next build",
    "start": "next start -p 3001",
    "lint": "next lint"
  },
```
# console.log()가 두 번 실행될때
[이 글](https://han-py.tistory.com/508)을 참고했습니다. 저는 next.config.mjs를 다음과 같이 작성해서 해결했습니다.
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: false,
};
  
export default nextConfig;
```
# 컨벤션
- 라우트 파일명은 소문자로 작성합니다.
# 테스트 코드
[참고](https://blog.pumpkin-raccoon.com/83)
# vscode extension
- ES7+ React/Redux/React-Native/JS...: 리액트 컴포넌트를 쉽게 생성할 수 있는 스니펫 제공
	- `rafce`: 리액트 컴포넌트 생성
- JavaScript and TypeScript Nightly
- Tailwind CSS IntelliSense