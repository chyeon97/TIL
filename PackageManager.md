# Package Manager


## 들어가며

회사 동기들과 `yarn berry`에 대해 알아보다가 `패키지매니저`와 `npm`의 의존성 관리에 대한 문제점을 알게 되어 이를 정리해본다.

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
    

## `npm`의 Linker
 
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
`redux`에서 사용하는 의존성을 파악한 뒤 `node_modules`가 있다면 또 그 밑으로 패키지를 추가해주는 과정으로 진행됩니다.


