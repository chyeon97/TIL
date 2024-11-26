# yarn berry 도입하기

<img src="./img/thumbnail/yarnberry.png" width="100%"/>

## 들어가며

`node_modules`를 벗어나고자 회사 동기와 함께 신규 프로젝트(next v14)에 `yarn berry` 를 도입하면서 부딪힌 오류와 해결방법을 정리해본다.

- yarn berry 설치
- module not found 오류 해결

## yarn berry 설치

본격적으로 설치하기 전에 기존 `npm` 패키지 매니저로 설치된 디렉토리를 삭제해준다.
```
    /node_modules
    /package-lock.json
```

아래와 같은 환경으로 설치할 것이다.
```
    yarn: v3.6.3
    typescript: 4.5.2
```


### 1. yarn berry 설치
```
    yarn set version 3.6.3
```

성공적으로 설치되었다면 `.yarn` 폴더와 `.yarnrc.yml` 파일이 생긴다.

### 2. yarnrc.yml 설정
```
    yarnPath: .yarn/releases/yarn-3.6.3.cjs
    nodeLinker: pnp
```

`nodeLinker`는 의존성을 관리하는 방식을 의미하며, 우리는 `pnp` 방식으로 사용했다.

### 3. yarn install

위에 설정한 환경으로 의존성을 설치한다.    
성공적으로 설치되면 `.pnp.cjs`와 `.pnp.loader.mjs`가 생긴다.

간단하게 `.pnp.cjs`를 읽어보면 패키지의 packageLocation으로 설치된 의존성 모듈의 위치를 파악할 수 있다.
> .yarn/cache에 zip으로 가지고 있다(pnp 방식)

```javascript
["eslint-plugin-react", [\
        ["npm:7.35.0", {\
          "packageLocation": "./.yarn/cache/eslint-plugin-react-npm-7.35.0-ce51a7759c-cd4d3c0567.zip/node_modules/eslint-plugin-react/",\
          "packageDependencies": [\
            ["eslint-plugin-react", "npm:7.35.0"]\
          ],\
          "linkType": "SOFT"\
        }],\
        ... 
      ]]

```


## module not found 오류 해결

위 방법으로 설치하고 나서 `module not found` 에러가 발생했다.

해당 문제는 `vscode`가 PnP + Typescript 환경의 yarn berry 패키지를 잘 인식하지 못해서 발생하는 문제였다.
그래서 `yarn dlx @yarnpkg/sdks vscode` 로 우회하여 주입시킨다.

그 다음, 아무 `.ts`에서 실행할 Typescript 버전을 설정해줘야한다.(최소 v4.5.2이상) 

1. `cmd + shift + p` 
2. typescript select 입력
3. 실행할 버전 선택 

프로젝트를 reload하거나 vscode를 껐다 해결된다.