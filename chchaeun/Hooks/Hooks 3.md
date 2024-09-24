# Hooks 3
## useOptimistic
UI를 낙관적으로 업데이트할 수 있게 해줌
- 인자로 주어진 일부 상태를 받아, 네트워크 요청과 같은 비동기 작업 기간 동안 달라질 수 있는 그 상태의 복사본을 반환
- 현재 상태와 작업의 입력을 취하는 함수를 제공하고, 작업이 대기 중일 때 사용할 낙관적 상태를 반환
### 매개변수
- state: 작업이 대기 중이지 않을 때 초기에 반환될 값
- updateFn(currentState, optimisticValue)
### 반환값
- optimisticState: 결과적인 낙관적 상태. 작업이 대기 중이지 않을 때는 state와 동일하지만 그렇지 않은 경우는 updateFn에서 반환된 값과 동일
- addOptimistic: 낙관적 업데이트가 있을 때 호출하는 디스패치 함수. 어떠한 타입의 optimisticValue라는 하나의 인자를 취하며, state와 optimisticValue로 updateFn을 호출
### vs Suspense
- Suspense: 선언적
- useOptimistic: 절차적
  - 좋아요 등 즉시 반영되면 좋은 상태를 다룰 때 유용할듯

## useReducer
컴포넌트에 reducer를 추가
### 매개변수
- action: 사용자에 의해 수행된 활동. 모든 데이터 타입 할당 가능. 컨벤션에 의해 action은 일반적으로 action을 정의하는 type 프로퍼티와 추가적인 정보를 표현하는 기타 프로퍼티를 포함한 객체로 구성
### 반환값
- dispatch 함수는 어떠한 값도 반환하지 않음
### 주의사항
- dispatch -> 오직 다음 렌더링에 사용할 state 변수만 업데이트. 호출 직후 변수 읽으면 최신화되지 않은 값 참조
- 이벤트 핸들러의 모든 코드가 수행되고 set함수가 모두 호출된 후 화면을 업데이트 -> 강제하고 싶으면 flushSync 사용

## useRef
렌더링에 필요하지 않은 값을 참조
### 매개변수
- initialValue: ref 객체의 current 프로퍼티 초기 설정값. 어떤 유형의 값이든 지정 가능. 초기 렌더링 이후부터는 무시
### 반환값
- current: 처음에는 전달한 initialValue로 설정, 나중에 다른 값을 바꿀 수 있음. ref 객체를 jsx 노드의 ref 어트리뷰트로 react에 전달하면 react는 current 프로퍼티를 설정
### 주의사항
- ref.current는 state와 달리 변이할 수 있음. 그러나 렌더링에 사용되는 객체(state의 일부)를 포함하는 경우 해당 객체를 변이해서는 안됨(적용이 안됨)
- ref.current 프로퍼티를 변경해도 react는 컴포넌트를 다시 렌더링하지 않음. ref는 일반 js 객체이기 때문에 react는 사용자가 언제 변경했는지 알지 못함
- 초기화를 제외하고는 렌더링 중 읽거나 쓰지 말것

## useState
컴포넌트에 state 변수를 추가할 수 있는 훅
### setState props 전달
- 부모의 상태를 자식이 변경하고 싶을 때 setState를 props로 그대로 넘기면 주도권을 넘기게 되기 때문에 위험하다고 생각
- 의미를 담은 액션함수를 만들어 전달
```js
// parent
export default function AppCounter(props) {
  const [count, setCount] = useState(0);
	
  const increase = (origin, amount) => {
	return origin + amount
  }

  const increaseCounter = (amount) => {
    setCount((prev) => increase(prev, amount));
  };

  return (
    <div className="container">
      <div className="banner">
        Total Count: {count} 
      </div>
      <div className="counters">
        <Counter total={count} increaseTotalCounter={(amount) => increaseCounter(amount)} />
		<Counter total={count} increaseTotalCounter={(amount) => increaseCounter(amount)} />
      </div>
    </div>
  );
}

// child
export default function Counter({total, increaseTotalCounter}) {
  const [count, setCount] = useState(0);

  const handleCountClick = () => {
      setCount((prev) => prev + 1);
      increaseTotalCounter(1);
  }

  return (
    <div className="counter">
      <span className="number">
        {count} <span className="total">/{total}</span>
      </span>
      <button
        className="button"
        onClick={handleCountClick}
      >
        Add +
      </button>
    </div>
  );
}
```

## useSyncExternalStore
- 외부 store를 구동할 수 잇는 훅
```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

1. subscribe 함수는 store를 구독하고 구독을 취소하는 함수를 반환해야 함
2. getSnapshot 함수는 store에서 데이터의 스냅샷을 읽어야 함

### 매개변수
- subscribe: 하나의 callback 인수를 받아 store에 구독하는 함수. store가 변경될 때, 제공된 callback이 호출되어 react가 getSnapshot을 다시 호출하고 필요한 경우 컴포넌트를 다시 렌더링하도록 해야. subscribe 함수는 구독을 정리하는 함수를 반환해야
- getSnapshot: 컴포넌트에 필요한 store 데이터의 스냅샷을 반환하는 함수. store가 변경되지 않은 상태에서 getSnapshot을 반복적으로 호출하면 동일한 값을 반환해야 함. 저장소가 변경되어 반환된 값이 다르면 컴포넌트 리렌더링
- getServerSnapshot?: store에 있는 데이터의 초기 스냅샷을 반환하는 함수. 서버 렌더링 도중과 클라이언트에서 서버 렌더링된 콘텐츠의 하이드레이션 중에만 사용. 
### 반환값
- 렌더링 로직에 사용할 수 있는 store의 현재 스냅샷
## useTransition
- UI를 차단하지 않고 상태를 업데이트할 수 있는 훅
- 컴포넌트 최상위 수준에서 useTransition을 호출하여 일부 state 업데이트를 transition으로 표시
### 반환값
1. isPending: 대기 중인 transition이 있는지
2. startTransition: 상태 업데이트를 transition으로 표시할 수 있게 해줌

