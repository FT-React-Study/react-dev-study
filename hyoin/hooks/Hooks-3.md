[Hooks-3](https://cooing-dust-8b6.notion.site/Hooks-3-0089dda2335a4cba9bee35d882133941?pvs=4)

### useOptimistic ( Canary )

- useOptimistic(state, updateFn) : UI를 낙관적으로 업데이트할수 있게해주는 hook ( canary )

  ```tsx
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

  ### 매개변수

  - `state` : 작업이 대기중이지 않을때 초기에 반환될 값
  - `updateFn(currentState, optimisticValue)` : 현재상태와 addOptimistic에 전달된 낙관적인 값을 취하는 함수. 순수함수. 반환값은 currentSate와 optimisticValue의 병합값

  ### 반환값

  - `optimisticState` : 결과적인 낙관적인 상태. 작업이 대기중이지 않을 때 `state` 와 동일, 그 외에는 `updateFn` 에서 반환된 값과 동일
  - `addOptimisitc` : 낙관적인 업데이트가 있을 때 호출하는 디스패치 함수.

### useReduce

- useReduce( reducer, initialArg, init? ): 컴포넌트에 reducer를 추가하는 react Hook, `useState`와 유사하지만, **업데이트 로직을 컴포넌트 외부의 단일함수(reducer)로 분리할 수 있다**는 차이가있음.

  ```tsx
  import { useReducer } from 'react';

  function reducer(state, action) {
    // ...
  }

  function MyComponent() {
    const [state, dispatch] = useReducer(reducer, { age: 42 });
    // ...
  ```

  ### 매개변수

  - `reducer` : state가 어떻게 업데이트되는지 지정하는 리듀서 함수. 순수함수이며 state와 action을 인수로 받아야하고, 다음 state를 반환해야한다.
  - `initialArg` : 초기 state가 계산되는 값.
  - 선택사항 `init` : 초기 state를 반환하는 초기화 함수.
    - 이 함수가 할당되지 않을 경우 초기 state는 `initialArg` 로 설정.
    - 할당되었을 경우는 초기값은 `init(initialArg)` 를 호출한 결과가 할당

  ### 반환값

  2개의 엘리먼트로 구성된 배열 반환

  - 현재 state
  - dispatch 함수

- Dispatch함수 : state 업데이트 & 리렌더링 유발. action을 인수로받음

  ```tsx
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  function handleClick() {
    dispatch({ type: 'incremented_age' });
    // ...
  ```

  ### 매개변수

  - action : 사용자에 의해 수행된 활동. 일반적으로 `type` 프로퍼티와 기타 프로퍼티를 가진 객체로 구성

  ### 반환값

  없음

  - `object.is` 비교를 통해 state 업데이트 여부를 확인.

### useRef

- useRef(initialValue) : 렌더링에 필요하지 않은 값을 참조할수 있는 hook

  ```tsx
  import { useRef } from 'react';

  function MyComponent() {
    const intervalRef = useRef(0);
    const inputRef = useRef(null);
    // ...
  ```

  ### 매개변수

  - `initialValue` : ref객체의 current 프로퍼티 초기값

  ### 반환값

  - `current` : 처음 전달한 `initialValue`로 설정됨. jsx 노드의 ref 어트리뷰트로 전달하면 current 프로퍼티를 설정한다.

- ref로 값 참조하기 → 값이 변경되도 리렌더링을 트리거하지 않는다.
  - counter, 스톱워치 등
- ref로 DOM 조작하기
- ref 컨텐츠 재생성 피하기 → ref는 처음에만 저장하고, 다음 렌더링부터는 무시.
  → video 객체와 같은 리소스가 많이 드는 객체를 ref로 초기화하는 방법도 있다.
  ```tsx
  function Video() {
    const playerRef = useRef(null);
    if (playerRef.current === null) {
      playerRef.current = new VideoPlayer();
    }
    // ...
  ```
- 커스텀컴포넌트의 경우에는 `forwardRef` 로 컴포넌트를 감싸서 부모 컴포넌트에서 ref를 insert해줄수 있음.

### useState

- useState(initialState) : 컴포넌트에 state 변수를 추가할 수 있는 hook

  ```tsx
  import { useState } from 'react';

  function MyComponent() {
    const [age, setAge] = useState(28);
    const [name, setName] = useState('Taylor');
    const [todos, setTodos] = useState(() => createTodos());
    // ...
  ```

  ### 매개변수

  - `initialState` : state 초기 설정값. 함수를 initialState로 전달할 경우 `초기화 함수`로 취급한다.
    - 이 때 함수는 순수함수이고
    - 인자를 받지 않아야하며
    - 어떤 값을 반환해야한다.

  ### 반환값

  두 개의 인자를 가진 배열을 반환

  - 현재 `state`
  - state를 업데이트하고, 리렌더링을 유발하는 `setState` 함수

- setState(nextState) 함수
  ### 매개변수
  - `nextState` : state가 될 값. 함수를 전달할 경우 `업데이터 함수`로 취급한다.
  ### 반환값
  없음
  - `object.is` 로 state 비교.
  - state 업데이트들을 batch로 관리. `flushSync` 를 통해 강제로 업데이트를 처리할 수도있다.
  - setState 함수는 다음 렌더링에서 반환할 useState에만 영향을 줌! → **state는 snapshot처럼 작동하기 때문**
  ### 업데이터함수
  - `대기중인 state`를 가져와서 `다음 state`를 계산..!
- 컴포넌트에 key를 전달해서 state를 초기화해 줄 수도 있다.

### useSyncExternalStore

- useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?) : 외부 store를 구독할수 있는 hook

  ```tsx
  import { useSyncExternalStore } from "react";
  import { todosStore } from "./todoStore.js";

  function TodosApp() {
    const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
    // ...
  }
  ```

  store에 있는 데이터의 스냅샷을 반환.

  **두개의 함수**를 인수로 전달한다.

  ### 매개변수

  - `subscribe` 함수 : 하나의 `callback` 인수를 받아 store에 구독하는 함수. 스토어가 변경되면 제공된 `callback`을 호출 해야하고, 호출되면 컴포넌트가 리렌더링된다.
    - `callback` 을 인자로 받음
    - 구독을 정리하는 함수를 반환해야한다.
  - `getSnapshot` 함수: 컴포넌트에서 필요한 store에서 데이터의 스냅샷을 반환
  - **선택사항** `getServerSnapshot` : store에 있는 데이터의 초기 스냅샷을 반환하는 함수. 서버렌더링 도중 & 하이드레이션 중에만 사용된다. 이 함수가 제공되지 않으면 서버에서 컴포넌트를 렌더링할 때 오류 발생.

  ### 반환값

  렌더링 로직에 사용할 수 있는 store의 현재 스냅샷

  - `getSnapshot` 이 반환하는 스냅샷은 불변이어야 한다.
  - 리렌더링하는 동안 다른 `subscribe` 함수가 전달되면 새로 전달된 함수를 사용해 store를 새로 구독함.

- 대부분의 리액트 컴포넌트 : props, state, context에서만 데이터를 읽음. but 외부 저장소에서 데이터를 읽어야하는 경우도 있음
  - 외부의 state를 보관하는 서드파티 라이브러리
  - 변경가능한 값을 노출하는 브라우저 API와 그 변경사항을 구독하는 이벤트
- Custom Hook으로 로직 추출해서 사용하는 경우가 많음
  ```tsx
  export function useOnlineStatus() {
    const isOnline = useSyncExternalStore(subscribe, getSnapshot);
    return isOnline;
  }
  ```

### useTransition

- useTransition() : UI 를 차단하지 않고 상태를 업데이트할 수 있는 hook. (non-blocking Transition)

  ```tsx
  import { useTransition } from "react";

  function TabContainer() {
    const [isPending, startTransition] = useTransition();
    // ...
  }
  ```

  ### 매개변수

  없음

  ### 반환값

  두개의 요소를 가진 배열을 반환

  - `isPending` 플래그 : 대기중인 Transition 이 있는지 여부
  - `startTransition` 함수 : 상태 업데이트를 Transition으로 표시할 수 있게 해주는 함수

- `startTransition` 함수: state 업데이트를 transition으로 표시할 수 있음

  ```tsx
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

  ### 매개변수

  - `scope` : 하나 이상의 `set` 함수 호출해서 일부 state를 업데이트하는 함수.
    - 리액트는 매개변수 없이 `scope`를 즉시 호출하고 호출 하는 동안 동기적으로 예약된 모든 state 업데이트를 Transition으로 표시한다.
    - 이는 `non-blocking` 이며 원치않는 로딩을 표시하지 않는다.

- state 업데이트를 non-blocking Transition으로 표시
- Transition에서 상위 컴포넌트 업데이트
- Transition 중에 보류 중인 시각적 state 표시
- 원치 않는 로딩 표시기 방지
- Suspense-enabled 라우터 구축
- (Canary) Error boundary로 사용자에게 오류 표시하기
