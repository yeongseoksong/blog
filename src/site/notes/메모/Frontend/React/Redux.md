---
{"dg-publish":true,"permalink":"//frontend/react/redux/","title":"Redux"}
---

#### useContext Vs Redux
---
useContext 와 Redux  둘 다 App 의 전역 상태를 다루기 위해 사용된다.  서로 같은 역할을 하는데 Redux가 등장한 이유는 useContext의 아래와 같은 단점들 때문이다.

- 대형 프로젝트에서 복잡한 구조가 발생한다.
```jsx
  return( 
		<AuthProvoider>
			<ThemeProvider>
				<AProvider>
					// ... 
					<YoutComponent/>
				</AProvider>
			</ThemeProvider>
		</AuthProvoider>
  )
```




- 위의 방법이 아니라면 , 모든 상태를 다루는 하나의 Context를 사용해야한다.
- useContext 는 자주 변경되는 데이터에 대해 성능이 좋지 않다.



#### Redux 동작 방식
---
![Pasted image 20240129111755.jpg](/img/user/0.%20%EC%9D%B4%EB%AF%B8%EC%A7%80/Pasted%20image%2020240129111755.jpg)
> Redux 아키텍쳐는 단방향으로만 데이터가 흐른다.


 - **Action** : 스토어 안에 있는 상태를 변화 하는 유일한 방법
 - **Reducer :**  누적 값을 저장해 새로운 값은 반환하는  함수형 도구인데 ,Redux 에서 Reducer 는 Action을 누적해 이전 상태와 계산해 새로운 상태를 반환한다. !== useReducer( )
 - **Store** : 전역상태를 저장하며 App 에 단 하나만 존재 해야한다. 
   [[책/쏙쏙 들어오는함수형 코딩/18. 반응형 아키텍처와 어니언 아키텍처\|18. 반응형 아키텍처와 어니언 아키텍처]] 링크와 같은 원리로 동작한다.

```javascript
const valueCell =(initialValue)=>{
	let currentValue= initialValue;
	let watchers=[]
	return {
		val(){  // redux 의 getStore
			return currentValue
		},
		update(callback){ //dispatch 
			const oldValue= currentValue;
			const newValue =callback(currentValue);
			if (oldValue!== newValue){
				currentValue=newValue;
				watchers.forEach(watcher=>watcher(newValue))
			}
		},
		addWatcher(callback){ //subscribe
			watchers.push(f)
		}
	}
}

```

위의 코드와 redux 의 차이점은 redux 는 initialValue 대신에 Reducer 를 사용한다. 위의 코드도 callback 함수를 순수 함수를 받지만 redux는 reducer 를 사용함으로써  action 만 disaptch 할 수 있고, 상태를 변경하는 함수가 reducer에 포함된다는 차이점이 있다.

```js
const redux = require("redux");

  

const counterReducer = (state = { counter: 0 }, action) => {
  if (action.type === "add") {
    return {
      counter: state.counter + action.value,
    };
  }
  if (action.type == "sub") {
    return {
      counter: state.counter - action.value
  }
  return state;
};

  

const store = redux.createStore(counterReducer);


console.log(store.getState()); //{counter:0}


const counterSub = () => {
  const latestState = store.getState();
  console.log(latestState);
};


store.subscribe(counterSub);
store.dispatch({ type: "add",value=1 }); //{counter:1}
store.dispatch({ type: "sub",value=2 }); //{counter:0}
```

#### React 에서 Redux 사용
___
React 에서 useSelector( )훅을 사용하면 해당 컴포넌트가 자동으로 Store 에 Subscribe( ) 된다. 
그러기 위해서는  'react-redux' 에서 제공하는 Provider를 적용하고자 하는 컴포넌트 상위에 선언해주어야한다. 아래는 최상위에 래핑 하도록 하겠다.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { Provider } from "react-redux";
import "./index.css";
import App from "./App";

// 선언한createStore(reducer) 을 export 한다.
import ReduxStore from "./store"; 
const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(
  <Provider store={ReduxStore}>
    <App />
  </Provider>

);
```

** useSelector**  : 컴포넌트에 subscribe 시킨다 .  컴포넌트가 삭제될 경우에 'react-redux' 에서 처리해준다.
** useDispatch** : Dispatch 함수를 사용할 수 있게해준다.

```jsx
import classes from "./Counter.module.css";

import { useSelector, useDispatch } from "react-redux";

const Counter = () => {
  const counter = useSelector((state) => state.counter);
  const dispatch = useDispatch();
  
  function addHandler() {
    dispatch({ type: "add", value: 1 });
  }
  function subHandler() {
    dispatch({ type: "sub", value: 1 });
   }
  

  return (
    <main>
      <h1>Redux Counter</h1>
      <div > {counter} </div>
      <div }>
        <button onClick={addHandler}>+</button>
        <button onClick={subHandler}>_</button>
      </div>
    </main>
  );
};

export default Counter;
```