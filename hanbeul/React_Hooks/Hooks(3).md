# 9주차. React Hooks (3)

## [useOptimistic](https://ko.react.dev/reference/react/useOptimistic)

_`useOptimistic` 훅은 현재 React Canary 채널에서만 사용 가능하다._

`useOptimistic`는 UI를 낙관적으로 업데이트 할 수 있게 해주는 Hook이다.

```js
const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

### 레퍼런스

> `useOptimistic(state, updateFn)`의 매개변수

- `state`: 작업이 대기 중이지 않을 때 초기에 반환될 값

- `updateFn(currentState, optimisticValue)`: 현재 상태와 addOptimistic에 전달될 낙관적인 값을 취하는 함수(순수함수).

> `useOptimistic`의 반환값

- `optimisticState`: 결과적인 낙관적인 상태. 작업이 대기 중이지 않을 때는 `state`와 동일, 그렇지 않을 경우 `updateFn`에서 반환된 값

- `addOptimistic`: 낙관적 업데이트가 있을 때 호출하는 디스페치 함수. `optimisticState`라는 하나의 인자를 취하며 `state`와 `optimisticValue`로 `updateFn`을 호출.

### 사용법

```js
import { useOptimistic } from "react";

function AppContainer() {
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    // updateFn
    (currentState, optimisticValue) => {
      // merge and return new state
      // with optimistic value
    }
  );
}
```

## [useReducer](https://ko.react.dev/reference/react/useReducer)

`useReducer`는 컴포넌트에 reducer를 추가하는 React Hook이다.

### 레퍼런스

> `useReducer(reducer, initialArg, init?)`의 매개변수

- `reducer`: state가 어떻게 업데이트 되는지 지정하는 리듀서 함수. 반드시 순수함수여야 한다. state와 action을 인수로 받아 다음 state를 반환.

- `initialArg`: 초기 state가 계산되는 값.

- optional `init`: 초기 state를 반환하는 초기화 함수. 이 함수가 없을 경우 초기 state는 initialArg로 설정. 할당되었다면 초기 state는 `init(initialArg)`를 호출한 결과가 할당됨.

> `useReducer`의 반환값

- `[state, dispatch]` 형태의 배열을 반환

- `state`: 첫번째 렌더링에서의 state는 `init(initialArg)` 또는 `initialArg`로 설정

- `dispatch`: state를 새로운 값으로 업데이트하고 리렌더링을 일으킴

### 사용법

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

> 실제 사용 예시

```js
import { useReducer } from "react";

function reducer(state, action) {
  if (action.type === "incremented_age") {
    return {
      age: state.age + 1,
    };
  }
  throw Error("Unknown action.");
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: "incremented_age" });
        }}
      >
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

- 버튼 클릭시 마다 `dispatch`로 `reducer` 함수 실행

- `dispatch`에는 `type`을 넣어줘 action을 구분할 수 있음

## [useRef](https://ko.react.dev/reference/react/useRef)

`useRef`는 렌더링에 필요하지 않은 값을 참조할 수 있는 React Hook이다.

### 레퍼런스

> `useRef(initialValue)`의 매개변수

- `initialValue`: ref의 `current` 프로퍼티 초기 설정 값. 초기 렌더링 이후 무시된다.

> `useRef`의 반환값

- 단일 프로퍼티를 가진 객체를 반환

- `current`: 처음에는 `initialValue`로 설정됨.

### 사용법

> ref로 값 참조

```js
import { useRef } from "react";

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

- 리액트의 렌더링 과정에서 `ref.current`를 쓰거나 읽으면 안된다.

- 대신 이벤트 핸들러나 Effect에서 사용.

> ref로 DOM 조작하기

```js
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

- 초기값은 null로 선언

> ref로 콘텐츠 재생성 피하기

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

- 일반적으로 렌더링 중에 ref.current를 쓰거나 읽는 것은 허용되지 않지만, 이 경우에는 결과가 항상 동일하고 초기화 중에만 조건이 실행되므로 충분히 예측할 수 있으므로 괜찮다.

## [useState](https://ko.react.dev/reference/react/useState)

`useState`는 컴포넌트에 state 변수를 추가할 수 있는 React Hook이다.

### 레퍼런스

> `useState(initialState)`의 매개변수

- `initialState`: state의 초기 설정값.

> `useState`의 반환값

- `[state, setState]` 형태의 배열을 반환

- `state`: 현재 state. 첫 번째 렌더링 중에는 `initialState`와 일치

- `setState`: state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 set 함수.

### 사용법

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
```

- `setState` 함수는 다음 렌더링에 대한 state 변수만 업데이트 한다. `setState` 함수를 호출한 후에도 state 변수에는 여전히 호출 전 화면에 있던 이전 값이 담겨있다(snapshot).

- `Object.is`를 활용해 기존 값과 비교. state가 동일할 경우 리렌더링을 유발하지 않음.

- React는 `setState` 업데이트를 batch 한다. 모든 이벤트 핸들러가 실행되고 `setState` 함수를 호출한 후 화면을 업데이트 함.

- 렌더링 도중 `setState` 함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 허용.

## [useSyncExternalStore](https://ko.react.dev/reference/react/useSyncExternalStore)

`useSyncExternalStore`는 외부 store를 구독할 수 있는 React Hook이다.

### 레퍼런스

> `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`의 매개변수

- `subscribe`: 하나의 `callback` 인수를 받아 store에 구독하는 함수. 스토어가 변경되면 제공된 `callback` 호출. `cleanUp` 함수를 반환해야 한다.

- `getSnapshot`: 컴포넌트에 필요한 store 데이터의 스냅샷을 반환하는 함수.

- optional `getServerSnapshot`: store에 있는 데이터의 초기 스냅샷을 반환하는 함수.

> `useSyncExternalStore`의 반환값

- 렌더링 로직에 사용할 수 있는 store의 현재 스냅샷

### 사용법

```js
import { useSyncExternalStore } from "react";
import { todosStore } from "./todoStore.js";

function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );
  // ...
}
```

- 가능하면 내장된 React state를 `useState` 및 `useReducer`와 함께 사용하는 것이 좋다.

- `useSyncExternalStore` API는 비 React 코드(브라우저 API 등)와 통합할 때 주로 유용함.

- 서버 렌더링을 사용하는 경우 서드 파티 데이터 저장소에 연결할 때 도움을 받을 수 있다.

## [useTransition](https://ko.react.dev/reference/react/useTransition)

`useTransition`은 UI를 차단하지 않고 상태를 업데이트 할 수 있는 React Hook이다.

### 레퍼런스

> `useTransition()`의 매개변수

- 매개변수를 받지 않음

> `useTransition`의 반환값

- `[isPending, startTransition]` 형태의 배열 반환

- `isPending`: 대기 중인 Transition 이 있는지 알려줌(boolean)

- `startTransition`: 상태 업데이트를 Transition 으로 표시할 수 있게 해주는 함수. `scope`를 매개변수로 받고, 아무것도 반환하지 않음.

  - `scope`: 하나 이상의 `set` 함수를 호출하여 일부 state를 업데이트 하는 함수

  - React는 `scope`를 즉시 호출하고, 호출하는 동안 동기적으로 예약된 모든 state 업데이트를 Transition으로 표시. 이는 non-blocking이며 원치 않는 로딩을 표시하지 않음

### 사용법

```js
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```
