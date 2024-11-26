# BABEL

## 들어가며
실무에서 Babel 버전을 올리면서 공부한 바벨의 개념과 부딪힌 문제에 대해서 정리해본다.

- 바벨이란 무엇인가?
- 바벨의 3딘계
- 바벨 설정 파일 차이
- 바벨 버전 올리면서 부딪힌 오류
- 참고문헌

## 바벨(BABEL)이란?
> Babel is a JavaScript compiler

1. 바벨은 리액트에서 사용하는 `JSX`와 같은 구문을 **`JS`로 변형해주는 도구**이다      
대표적으로 React는 `JSX`를 사용하지만, `npm run build`를 해보면 `js`로 바뀌어있다.
또한 `ts` 구문을 `js`로 바꿔준다.


```javascript
// JSX 
export default function DiceRoll(){
  const getRandomNumber = () => {
    return Math.ceil(Math.random() * 6);
  };

  const [num, setNum] = useState(getRandomNumber());

  const handleClick = () => {
    const newNum = getRandomNumber();
    setNum(newNum);
  };

  return (
    <div>
      Your dice roll: {num}.
      <button onClick={handleClick}>Click to get a new number</button>
    </div>
  );
};


// BABEL Result
export default function DiceRoll() {
  const getRandomNumber = () => {
    return Math.ceil(Math.random() * 6);
  };
  const [num, setNum] = useState(getRandomNumber());
  const handleClick = () => {
    const newNum = getRandomNumber();
    setNum(newNum);
  };
  return /*#__PURE__*/React.createElement("div", null, "Your dice roll: ", num, ".", /*#__PURE__*/React.createElement("button", {
    onClick: handleClick
  }, "Click to get a new number"));
}
;
```


2. 바벨은 최신 자바스크립트 문법을 구형 브라우저에서도 동작할 수 있도록 해주는 도구이다.   

## 바벨 3단계
1. 파싱(Parsing)
    - 소스를 읽어서 AST(추상 구문 트리)로 변환
    - `@babel/parser` 가 대표적이다.

2. 변환(Transformation)
    -  규칙(plugin, presset)에따라 AST를 변형하는 작업 
    - presset은 플러그인의 모음집


3. 출력(Printing)
    - AST를 다시 소스로 변환하는 단계

## 바벨 설정 파일
바벨을 설정하는 파일 형식은 `babelrc`와 `babel.confign.json`이 있다.   
실무에서 기존에 `babelrc`를 사용하고 있었고 구글링을 해보니 설정파일에 따른 차이점이 있었다.

| babelrc      | 차이점            | babel.confign.json                      |
|-------------|--------------|---------------------------|
| 파일 단위            |  범위 | 프로젝트 전체  |
| 특정 디렉토리나 파일에만 적용할 경우에 사용           |  사용목적 | 프로젝트에 동일한 환경의 바벨 설정을 할때 사용  |



## 바벨 버전 업그레이드 중 부딪힌 오류

| 오류 구분                   | 오류 내용                                                                                                                                                   | 설명                                   | 해결법                                                                                      |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|------------------------------------------------------------------------------------------|
| `BABEL_TRANSFORM_ERROR` | "Property arguments[0] of CallExpression expected node to be of a type ["Expression","SpreadElement","ArgumentPlaceholder"] but instead got "JSXIdentifier" | 바벨이 JSX 구문을 제대로 인식하지 못해서 발생          | `transform-react-inline-elements` -> `@babel/plugin-transform-react-inline-elements` 로 변경 |
| `BABEL_PARSE_ERROR`     | "VarRedeclaration"                                                                                           | 구문 오류 - `var`로 선언한 변수가 재선언 되어 발생하는 오류 | 직접 코드 수정                                                                                 |
| `BABEL_PARSE_ERROR`     | "RestTrailingComma"                                                                                           | 구문 오류 - 객체/배열 마지막 요소에 콤마를 허용하지 않음    | 직접 코드 수정                                                                                 |

## 참고문헌
https://babeljs.io/docs/#babel-is-a-javascript-compiler
https://babeljs.io/docs/configuration