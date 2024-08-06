# Package Manager

<img src="./img/thumbnail/패키지매니저_썸네일.png" width="100%"/>


## 들어가며

회사 동기들과 `yarn berry`에 대해 알아보다가 `패키지매니저`와 `npm`의 의존성 관리에 대해 학습한 내용을 정리해본다.

- 패키지 매니저와 동작 원리
- npm의 의존성 관리

## 패키지 매니저란?
> 패키지를 다루는 작업을 편리하고 안전하게 수행하기 위해 사용되는 툴

쉽게 말하자면, `import`문이나 `require()`를 사용하여 외부 의존성을 참조하는 것을   
올바르게 참조할 수 있도록 보장해주는 역할을 수행하는 것이다.

```javascript
import React from 'react';
require('react');
```


## 패키지 매니저의 동작 단계

패키지 매니저는 `Resolution` - `Fetch` - `Link` 단계로 동작한다.

1. Resolution 단계
   - `package.json`을 읽어 명시된 버전 범위에 따라 정확한 버전을 결정한다.
   - 의존성의 의존성을 확인하고 버전을 결정하여 `package-lock.json`을 작성한다.

2. Fetch 단계
    - 앞서 Resolution 단계에서 결정된 버전을 실제로 다운로드한다.
    - 99%는 npm 레지스트리에서 가져온다.

3. Link 단계
    - 앞서 Fetch 단계에서 가져온 라이브러리를 소스에서 사용할 수 있는 환경을 제공한다.
    - `npm`, `pnmp`, `PnP`등 패키지 매니저 종류에 따라 차이가 있다.
    

## `npm` 의존성 관리
 
`package.json`에 있는 모든 의존성을 `node_modules`에 하나씩 디렉토리를 구성하는 방법

```
my-service/
└─ node_modules/
|  ├─ react/
|  |  
|  └─ redux/
|     └─ node_modules/
|         └─ symbol-observable
|
└─ src
    └─ index.ts
```

예를 들어, 위 구조를 보면 `my-service` 프로젝트에서 `react`와 `redux`를 사용한다면,   
`my-service`의 `node_modules`에 `react`와 `redux` 패키지를 추가하고   
`redux`에서 사용하는 의존성을 파악한 뒤 `node_modules`가 있다면 또 그 밑으로 패키지를 추가해주는 과정으로 진행된다.

이러한 `node_modules`는 여러 단점이 있다.

첫째, **비효율적인 의존성 검색**


```
$ node
> require.resolve.paths('react')
[
  '/Users/bagchaeyeon/Documents/study/book-mark/repl/node_modules',
  '/Users/bagchaeyeon/Documents/study/book-mark/node_modules',
  '/Users/bagchaeyeon/Documents/study/node_modules',
  '/Users/bagchaeyeon/Documents/node_modules',
  '/Users/bagchaeyeon/node_modules',
  '/Users/node_modules',
  '/node_modules',
  '/Users/bagchaeyeon/.node_modules',
  '/Users/bagchaeyeon/.node_libraries',
  '/usr/local/lib/node',
  '/Users/bagchaeyeon/.node_modules',
  '/Users/bagchaeyeon/.node_libraries',
  '/usr/local/lib/node'
]

```

토이 프로젝트로 개발하고 있는 디렉토리에서 `require()`문으로 `react` 패키지를 불러오는 상황일 때,   
위에 보이는 목록처럼 `npm`은 패키지를 찾기위해 계속 상위 디렉토리의 `node_modules`를 탐색한다.

즉, 의존성이 깊어질 수록 패키지를 탐색하는 속도가 현저히 떨어지게 된다.

둘째, **유령 의존성**

`npm`은 호이스팅(Hoisting) 기법으로 의존성을 관리한다.

<img src="./img/npm.png" width="100%"/>


왼쪽 트리에서 `A(1.0)`과 `B(1.0)` 패키지는 두 번 설치해야하므로 디스크 공간 낭비가 발생한다.
이러한 트리를 `npm`과 `yarn`은 오른쪽 트리처럼 바꾼다.

따라서, `package-1`에서는 원래 `B(1.0)`패키지를 참조할 수 없었지만, 참조할 수 있게 된다.   
이와 같이 **직접 의존하고 있지 않은 라이브러리를 참조할 수 있는 현상**을 `유령 의존성(Phantom Dependency)`이다.

실제로 `npm`패키지를 사용하는 회사 `node_moduels`에서도 이러한 현상을 볼 수 있었다.

```
company-service/
└─ node_modules/
|  ├─ react/
|  ├─ core-js/
|  |
|  └─ recharts/
|  |  └─ node_modules/
|  |      └─ core-js/
|  |      └─ classnames/
|  |      └─ react-smooth/
|  |      └─ ...
|  |
|  └─ react-tap-event-plugin/
|     └─ node_modules/
|         └─ core-js/
|         └─ ...
|
└─ src

```

`company-service`의 `package-json`에는 `core-js`에 대해 명시하지 않았다.   
하지만 `react-tap-event-plugin`와 `recharts` 패키지 내부에 `core-js`를 사용하기 때문에   
최상단 `node_moduels`에 설치되었다.