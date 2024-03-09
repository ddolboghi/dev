# Next.js 란?
- 풀스택 웹 애플리케이션을 구축하기 위한 리액트 프레임워크입니다.
- convention-over-configuration이라는 소프트웨어 설계 패러다임 기반으로 만들어졌습니다.

> [!tip] URL segment
> ![[url-segment.png]] 
# Next.js의 서버 사이드 렌더링(SSR)과 정적사이트 생성(SSG)
- 서버 사이드 렌더링: 각 요청에 대해 서버에서 모든 리소스를 렌더링한 후 보냅니다. 이를 통해 SEO 개선, 퍼포먼스 개선, 보안 향상이 가능해집니다.
  단, CSR보다 많은 컴퓨팅 자원을 소모하고 페이지 요청 처리 시간도 길어집니다.
> [!warning] SSR에서 말하는 서버는 프론트엔드의 서버입니다.

![[nextjs-ssr.png]]
1. 먼저 서버에서 주어진 페이지의 모든 데이터를 가져옵니다.
2. 서버는 페이지의 HTML을 렌더링합니다.
3. 페이지의 HTML, CSS, JavaScript가 클라이언트에 전송됩니다.
4. 생성된 HTML 및 CSS를 사용하여 상호 작용하지 않는 사용자 인터페이스가 표시됩니다.
5. 마지막으로, React는 사용자 인터페이스를 [하이드레이트(hydrate)](https://react.dev/reference/react-dom/client/hydrateRoot#hydrating-server-rendered-html)하여 상호 작용하게 만듭니다.
- **모든 데이터가 가져와진 후에만 서버에서 페이지의 HTML을 렌더링 할 수 있습니다.** 
- 클라이언트에서는 페이지의 모든 컴포넌트 코드가 다운로드된 후에만 React가 UI를 가져와 처리 할 수 있습니다.
# Next.js의 주요 기능 및 장점
- 자동 코드 분할: 각 페이지에서 사용되는 컴포넌트를 기반으로 애플리케이션 코드를 자동으로 작은 번들로 분할해 사용자가 현재 페이지에 필요한 코드만 다운로드되도록 합니다. 이를 통해  로딩 시간이 단축되고 성능이 향상됩니다.
- 핫 모듈 교체(HMR): 개발 시 전체 페이지를 새로고침하지 않아도 애플리케이션의 변경 사항을 확인할 수 있습니다.
- 자동 페이지 라우팅: `app` 디렉토리의 파일 구조를 기반으로 자동으로 페이지 라우팅을 처리합니다.
> [!페이지 라우팅]
> - 웹 애플리케이션에서 사용자가 요청한 URL 경로에 따라 적절한 페이지를 표시하는 기술 또는 프로세스입니다.
> - 페이지 라우팅을 통해 SPA(Single-Page Application)는 사용자가 애플리케이션 내에서 여러 페이지를 탐색하고 상호작용할 수 있도록 합니다.

- 하이브리드 렌더링: 서버 사이드 렌더링과 정적 사이트 생성 모두 지원해 특정 방식을 선택하거나 하나의 애플리케이션 내 두 기술을 결합할 수 있습니다.
- hydration: 서버에서 렌더링한 DOM과 클라이언트가 렌더링한 DOM이 한데 섞여 싱글 페이지 애플리케이션처럼 보이게 합니다. 
- [Next.js 사용 시 직면할 수 있는 함정, 그리고 이를 극복하는 전략](https://reactnext-central.xyz/blog/nextjs-challenges)

> [!warning] **컴포넌트는 익명함수로 만들 수 없습니다.**
> \[ eslint ERROR ] Component definition is missing display nameeslintreact/display-name

> [!tip] **클라이언트가 서버컴포넌트를 import하면 서버 컴포넌트가 클라이언트 컴포넌트가 됩니다.** 
> 서버컴포넌트가 클라이언트컴포넌트를 임포트하는건 아무 영향 없습니다.
> 서버컴포넌트를 import가 아니라 props로 전달하면 서버컴포넌트로 쓸 수 있습니다.
## 파일 규칙

| 파일명                                                                                                                    | 설명                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| [`layout`](https://reactnext-central.xyz/docs/nextjs/routing/pages-and-layouts#%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83)   | 세그먼트와 그 자식에 대한 공유 UI                                                                  |
| [`page`](https://reactnext-central.xyz/docs/nextjs/routing/pages-and-layouts#%ED%8E%98%EC%9D%B4%EC%A7%80)              | 라우트의 고유 UI 및 라우트를 넣어 URL로 접근 가능하게 함                                                   |
| [`loading`](https://reactnext-central.xyz/docs/nextjs/routing/loading-ui-and-streaming)                                | 세그먼트와 그 자식에 대한 로딩 UI                                                                  |
| [`not-found`](https://nextjs.orghttps//nextjs.orghttps://nextjs.org/docs/app/api-reference/file-conventions/not-found) | 세그먼트와 그 자식에 대한 404 UI                                                                 |
| [`error`](https://reactnext-central.xyz/docs/nextjs/routing/error-handling)                                            | 세그먼트와 그 자식에 대한 에러 UI                                                                  |
| [`global-error`](https://reactnext-central.xyz/docs/nextjs/routing/error-handling)                                     | 전역 에러 UI                                                                              |
| [`route`](https://reactnext-central.xyz/docs/nextjs/routing/route-handlers)                                            | 서버 사이드 API 엔드포인트                                                                      |
| [`template`](https://reactnext-central.xyz/docs/nextjs/routing/pages-and-layouts#%ED%85%9C%ED%94%8C%EB%A6%BF)          | 특별하게 다시 렌더링되는 레이아웃 UI                                                                 |
| [`default`](https://nextjs.orghttps//nextjs.orghttps://nextjs.org/docs/app/api-reference/file-conventions/default)     | [병렬 라우트](https://reactnext-central.xyz/docs/nextjs/routing/parallel-routes)를 위한 기본 UI |

> **참고**: 이 파일들에 `.js`, `.jsx` 또는 `.tsx` 확장자를 사용할 수 있습니다.

## 컴포넌트 계층 구조
- 라우트 세그먼트의 특별한 파일에 정의된 React 컴포넌트는 특정 계층 구조에서 렌더링됩니다:
	- `layout.js`
	- `template.js`
	- `error.js` (React 에러 경계)
	- `loading.js` (React 서스펜스 경계)
	- `not-found.js` (React 에러 경계)
	- `page.js` 또는 중첩된 `layout.js`

- 중첩된 라우트에서는 세그먼트의 컴포넌트가 부모 세그먼트의 컴포넌트 내부에 중첩됩니다.
## 코드 공동 위치(Colocation)
- 특별한 파일 외에도 `app` 디렉토리 내의 폴더 안에 본인의 파일들(컴포넌트, 스타일, 테스트 등)을 함께 위치시킬 수 있습니다.
  이는 폴더가 라우트를 정의하는 반면, `page.js` 또는 `route.js`에서 반환된 내용만 URL로 접근 가능하기 때문입니다.
	![[nextjs-colocate.png]]
# `console.log()` 출력 위치
- 클라이언트 컴포넌트는 브라우저 콘솔창에 출력됩니다.
- 서버 컴포넌트는 터미널에 출력됩니다. 