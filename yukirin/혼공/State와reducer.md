> 제가 아기 시절... "state와 reducer의 차이가 뭐냐" 라는 물음을 받은 적이 있습니다.
> 그 때, 저는 reducer==redux인 줄 알고 state와 redux의 차이를 말해버렸습니다. (ㅋㅋ)
>
> useReducer를 보니 그때의 악몽이 떠올라서 정리해봅니다. "state와 reducer 비교하기" 🫠👍

<br/>
<br/>

```javascript
const nextState = reducer(state, { type: 'incremented_age' });
dispatch({ type: 'incremented_age' });
console.log(nextState); // { age: 43 } 출력

const [state, setState] = useState({ age: 42 });
setState({ age: 43 });
console.log(staet); // { age: 42 } 출력
```
React의 State와 reducer는 기본적으로 비슷한 컨셉에 기반하고 있지만 **상태 업데이트 시점**과 **상태 업데이트 처리 방식**에서 차이가 있다. 때문에 위 예시와 같이 출력되는 값이 다른 걸 확인할 수 있다.

따라서 이 차이를 이해하기 위해, React의 State와 reducer 함수를 구분하고자 한다!

<br/>

### 스냅샷으로 관리되는 State
React의 State는 컴포넌트가 렌더링될 때마다 **스냅샷**으로서 동작한다. 즉, 한 번 렌더링된 상태는 변경되지 않고, 새로운 상태는 `setState` 함수가 호출될 때 갱신된다.
- State가 변경되면 다음 렌더링에서 그 변경 사항이 반영된다.
- 업데이트가 즉시 이루어져도, 현재 렌더링에서 바로 반영되는 것이 아닌 다음 렌더링에서 반영된다.
- 그래서 `setState` 후에 바로 값을 참조하면, 업데이트된 값이 아닌 이전 상태가 보인다.

이는 **렌더링 주기를 제어하여 성능을 최적화하고, 불필요한 렌더링을 방지**하고자 하는 React의 설계 방식이다.

<br/>

### reducer 함수의 동작
reducer는 React 외부에서 실행되는 순수 함수로서, 현재 상태와 액션을 받아 새로운 상태 객체를 **즉각적으로 계산하여 반환**한다.
- reducer 내부에서는 상태가 즉시 변경되며, 변경된 상태는 바로 사용할 수 있다.
- reducer 함수는 기존의 상태를 직접 변경하지 않고 새로운 상태 객체를 반환한다.

<br/>

### 차이점: 비동기 vs 동기
둘의 차이를 정리하자면, **State는 비동기**적으로 업데이트가 처리되며, **reducer는 동기**적으로 업데이트가 처리된다.
- State는 배치 업데이트를 통해 한 번의 렌더링 주기 내에서 여러 업데이트를 모아서 처리된다.
- reducer는 상태를 즉각 반영하고 `useReducer` 훅을 통해 React의 상태 관리 흐름에 연결된다.
- 다만 `useReducer`는 State처럼 배치 업데이트 방식으로 동작하기 때문에 아래와 같은 결과를 볼 수 있다.
```javascript
const [state, dispatch] = useReducer(reducer, { age: 42 });
const nextState = reducer(state, { type: 'incremented_age' });
dispatch({ type: 'incremented_age' });

console.log(state);     // { age: 42 } 출력
console.log(nextState); // { age: 43 } 출력
```
즉, `useReducer`도 여러 업데이트를 한 번에 렌더링하도록 하고, 핵심은 reducer는 즉시 상태를 업데이트 한다는 것!

<br/>

### reducer와 렌더링 연결
reducer 업데이트는 React의 외부에서 수행되며 즉각 업데이트 된다고 했는데, `useReducer`와 만나면 State같이 동작한다. 내부에서 어떻게 상태 관리를 할까? 아래는 업데이트 처리 과정이다.
- `useReducer` 훅이 처음 호출될 때, React는 `reducer` 함수와 함께 제공된 초기 상태를 저장하고 이는 React의 내부 상태 관리 시스템에 저장된다.
- `dispatch`가 호출되면, React는 **업데이트가 필요함을 인식**하고 현재 상태와 전달된 액션을 `reducer`에 넘긴다.
- `reducer`는 새로운 상태 객체를 즉시 반환한다.
- 이 새로운 상태는 React 내부에 저장되고, 상태 변경 가능성이 감지되어 **리렌더링이 트리거**된다.
- React는 기존 상태와 새로운 상태를 비교하고 상태가 변경되었다면, 컴포넌트를 리렌더링한다.

<br/>

-----

<br/>

리액트로 개발을 하다 보면 컴포넌트의 확장성을 고민하게 된다. 이럴 때 **IoC(Inversion of Control) 즉, 제어 역전 패턴**을 통해 컴포넌트를 사용하는 개발자에게 컴포넌트의 제어권을 넘겨줌으로써 사용자가 원하는 대로 컨트롤할 수 있도록 한다.


### State Reducer 패턴
State Reducer 패턴은, 컴포넌트 내부의 상태와 해당 상태의 변화를 처리하는 로직들을 컴포넌트 외부에서 컨트롤할 수 있는 방안에 대한 패턴이다.
```typescript
type CounterActionType = "INCREMENT" | "DECREMENT" | "CHANGE";

interface CounterAction {
  type: CounterActionType;
  value?: any;
}

const CounterReducer: Reducer<number, CounterAction> = function (state = 0, action) {
  switch (action?.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "CHANGE":
      return action.value ?? 0;
    default:
      return state;
  }
};

const Counter: React.FC<ICounterProps> = function () {
  const [count, dispatch] = useReducer(CounterReducer, 0);

  return (
    <div>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <input value={count} onChange={event => dispatch({ type: "CHANGE", value: Number(event.target.value) })} />
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
    </div>
  );
};
```
위 예제의 컴포넌트의 특이한 점은 `CounterReducer`와 `useReducer`를 사용해서 컴포넌트의 상태를 관리하고 있다는 것이다. 이 컴포넌트의 구조를 좀만 수정하면 reducer를 외부에서 정의해서 props로 넘겨줄 수 있다.

```typescript
/** StateReducerCounter.tsx */

// 외부에서 정의할 리듀서의 형태
// state,action 처리뿐만 아니라 next 함수 호출을 통해 내부 리듀서를 사용할 수 있다.
export type OuterReducer = (state: number, action: CounterAction, next?: typeof CounterReducer) => number;

// 외부에서 정의한 리듀서와 내부 리듀서 결합 함수
function composeReducer(outerReducer?: OuterReducer): Reducer<number, CounterAction> {
  return function (prevState, action) {
    if (!outerReducer) {
      return CounterReducer(prevState, action);
    }

    return outerReducer(prevState, action, CounterReducer);
  };
}

// Counter Component
interface ICounterProps {
  reducer?: outerReducer;
}

const Counter: React.FC<ICounterProps> = function ({ reducer }) {
  // 외부 리듀서 + 내부 리듀서
  const [count, dispatch] = useReducer(composeReducer(reducer), 0);

  return (
    <div>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <input value={count} onChange={event => dispatch({ type: "CHANGE", value: Number(event.target.value) })} />
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
    </div>
  );
};
```
```typescript
/** App.tsx */
import Counter, { OuterReducer } from "./index";

const counterReducer: OuterReducer = function (state, action, next) {
  // INCREMENT 액션만 수정, 나머지 액션들은 내부 리듀서(next) 사용
  switch (action.type) {
    case "INCREMENT":
      return state + 2;
    default:
      return next?.(state, action) ?? 0;
  }
};

function App() {
  return (
    <div style={{ padding: 40 }}>
      <Counter reducer={counterReducer} />
    </div>
  );
}
```
위 예제의 `<Counter/>` 컴포넌트는 props로 `{reducer}`를 전달받고, 컴포넌트 내부에서 사용하던 `CounterReducer`와 결합해서 사용한다. 
**컴포넌트 사용자는 원하는 대로 동작하는 새로운 reducer를 작성하여 프로퍼티로 넘겨줌으로써, 해당 컴포넌트를 컨트롤**할 수 있다.

이러한 방식을 사용하면 이제 더 이상 컴포넌트의 프로퍼티에 대해 신경 쓸 필요가 없다.
컴포넌트의 콜백 함수가 변경되거나 새로운 콜백 함수가 추가된다고 하더라도 결국 `{reducer}`프로퍼티만 전달하면 되기 때문에, 사용자는 훨씬 간결하게 컴포넌트를 사용할 수 있다.
또한 `reducer`는 `useState`나 `useEffect`같은 훅을 사용하지 않기 때문에 굳이 컴포넌트 내부에서 작성하지 않고 별도로 분리해서 작성할 수 있다!
