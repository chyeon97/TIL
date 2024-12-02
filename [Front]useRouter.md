# BABEL
<img src="./img/thumbnail/nextjs.png" width="100%"/>

## 들어가며
실무에서 Next 내 `useRouter` 훅을 사용하면서 부딪힌 오류에 대해 정리해본다.

- useRouter란 무엇인가
- App Router에서의 useRouter 사용기
- Page Router의 useRouter vs App Router의 useRouter


## useRouter란?
> 현재 브라우저의 `pathname`, `basePath` 등 URL 정보를 접근할 떄 사용하는 훅

```javascript
import { useRouter } from 'next/router'
 
 // 현재 URL과 동일한 a태그 글씨 색상 변경 예제
function ActiveLink({ children, href }) {
  const router = useRouter()
  const style = {
    color: router.asPath == href ? 'red' : 'black',
  }
 
  const handleClick = (e) => {
    e.preventDefault()
    router.push(href)
  }
 
  return (
    <a href={href} onClick={handleClick} style={style}>
      {children}
    </a>
  )
}
 
export default ActiveLink
```

위 코드에서 **const router = useRouter()** 왜 이렇게 선언해서 사용할까? 
그 이유는 `useRouter`가 `Object` 타입의 데이터를 반환하기 때문이다.
`useRouter`가 가진 데이터는 아래를 포함해 [여러가지](https://nextjs.org/docs/pages/api-reference/functions/use-router)가 있다. 

| 명칭 | 자료형 |  설명 |
|-----|-----|-------|
| `pathname` | string | 현재 경로 파일의 경로로, `/pages 디렉토리` 이후의 경로를 나타냅니다.|
|`query`| Object | 쿼리 문자열을 객체로 변환한 값으로, 동적 경로 매개변수를 포함합니다. | 
| `asPath` | string | 브라우저에 표시된 경로로, 검색 파라미터(search params)를 포함합니다.| 
| `basePath` | string | 기본경로를 의미합니다. |
|`locale` | string | 현재 활성화된 언어가 반환됩니다. |


## AppRouter에서의 useRouter 사용하기

앞서 정리한 `useRouter`는 Pages Router에서만 사용할 수 있는 내용이다.   
다시 말하자면, App Router에서는 사용할 수 없는 내용이다. 

<img src="./img/nextrouter error.png" alt="오류"/>

위 오류는 내가 직접 부딪힌 내용의 오류이며, 다음과 같은 내용이다.
> App Router가 `next/router`의 `useRouter`와 다른 동작 방식을 가지고 있다.


## Page Router의 useRouter vs App Router의 useRouter

1. Page Router
    - 기존 라우팅 방식이고 `useRouter`는 `next/router`에서 가져온다.
    - 클라이언트 중심 라우팅이기에(= 라우터가 클라이언트에서 작동함) Javascript가 필요하다.
    - `RouterContext`가 필요하며, Next.js가 페이지 렌더링 시 자동으로 설정한다.

2. App Router
    - 클라이언트 컴포넌트(`use client`)환경에서 `use Router`를 사용할 수 있으며, `next/navigation`에서 가져온다.
    - `use Router`는 경로 정보를 제공하지 않으며, 오직 라우팅 동작을 위한 메서드가 있다. ex) `push()`, `replace()`, `back()`
    - 경로 정보를 제공받기 위해서는 `usePathname`, `useSearchParams`와 같이 따로 훅을 사용해야 한다.
    - `RouterContext`가 존재하지 않는다.


위 두가지 차이점을 비교하면서 내가 부딪힌 문제의 원인에 대해 알 수 있었다.

<em>NextRouter was not mouonted.</em>  

Pages Router의 useRouter는 RouterContext가 필요하지만, App Router에는 해당 컨텍스트가 없습니다.
> App Router에서의 `useRouter`는 `next/navigation`에서 가져오자!


