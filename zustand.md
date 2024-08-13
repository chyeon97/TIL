# zustand

<img src="./img/thumbnail/zustand.jpg" width="100%"/>

## 들어가며

토이 프로젝트로 상태 관리 라이브러리로 zustand를 도입하면서 학습한 사용법과 사용하면서 느낀 장단점에 대해 정리해본다.

- zustand란?
- 사용법
- zustand 코어 파헤쳐보기

## zustand란?
> Redux, Recoil과 같은 상태 관리 라이브러리이다.
- Hook과 Typescript를 기반하여 만들어졌다.
- `Flux 패턴`의 컨셉

## 사용법
> 다른 상태관리 라이브러리와 다르게 store만 생성해주면 바로 사용이 가능하다.

#### 1. 설치   
```
npm install zustand
```

####  2. store 생성
> create 함수를 이용하여 상태와 액선을 정의하면 Hook으로 리턴한다.
>> set 함수를 통해 상태값을 변경할 수 있다.
```javascript
    import { create } from 'zustand'
    
    const useStore = create((set) => ({
      bears: 0, // 상태
      increasePopulation: () => set((state) => ({ bears: state.bears + 1 })), // 액션
      removeAllBears: () => set({ bears: 0 }), // 액션
      updateBears: (newBears) => set({ bears: newBears }), // 액션
    }))
```

#### 3. 사용
> 훅을 사용할 떄, 어떤 형태로 꺼낼지 선택하여(Selector) 사용하거나, 그렇지 않으면 store 전체가 리턴된다.

```javascript
// 상태를 꺼낸다
function BearCounter() {
  const bears = useStore(state => state.bears);
  return <h1>{bears} around here ...</h1>;
}

// 액션을 꺼낸다
function Controls() {
  const increasePopulation = useStore(state => state.increasePopulation);
  return <button onClick={increasePopulation}>one up</button>;
}

// 전체를 꺼낸다
function AllControls() {
    const {bears, increasePopulation, removeAllBears, updateBears} = useStore(state => state)   
    return <div>...</div>
} 

```
### zustand 파헤쳐보기

스토어를 생성할때 사용하는 `set`은 무엇일까?   
`create`함수 를 파헤쳐보자

```javascript
// 함수 최하단부에 언급되지만 set, get 함수와
// `create` 함수를 통해 만들어지는 내부 API를 인자로 전달받을 수 있다.
export function create(createState) {
  // 스토어의 상태는 클로저로 관리된다.
  let state;

  // 상태 변경을 구독할 리스너를 Set으로 관리한다.
  // 배열로 관리할 경우 중복된 리스너를 솎아내기 어렵기 때문이다.
  const listeners = new Set();

  const setState = (partial, replace) => {
    const nextState = typeof partial === 'function' ? partial(state) : partial;
    if (nextState !== state) {
      const previousState = state;

      state = replace ? nextState : Object.assign({}, state, nextState);

      listeners.forEach(listener => listener(state, previousState));
    }
  };

  const getState = () => state;

  const subscribeWithSelector = (
    listener,
    selector = getState,
    equalityFn = Object.is
  ) => {
    let currentSlice = selector(state);

    function listenerToAdd() {
      const nextSlice = selector(state);
      if (!equalityFn(currentSlice, nextSlice)) {
        const previousSlice = currentSlice;
        listener((currentSlice = nextSlice), previousSlice);
      }
    }

    listeners.add(listenerToAdd);
    return () => listeners.delete(listenerToAdd);
  };

  const subscribe = (listener, selector, equalityFn) => {
    if (selector || equalityFn) {
      return subscribeWithSelector(listener, selector, equalityFn);
    }
    listeners.add(listener);
    return () => listeners.delete(listener);
  };

  // 모든 리스너를 제거한다. 하지만 이미 정의된 상태를 초기화하진 않는다.
  const destroy = () => listeners.clear();

  const api = { setState, getState, subscribe, destroy };

  // 인자로 전달받은 createState 함수를 이용하여 최초 상태를 설정한다.
  state = createState(setState, getState, api);

  return api;
}

```