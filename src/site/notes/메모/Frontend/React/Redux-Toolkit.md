---
{"dg-publish":true,"permalink":"//frontend/react/redux-toolkit/","title":"Redux-Toolkit"}
---

Redux Store 의 Reducer 하나에서 여러 상태를 관리하면 Reducer 의 구조가 복잡해진다. 그렇기 때문에 Redux-toolkit 은 createSlice( )를 제공해 상태에 해당하는 reducer 조각들을 생성하고 configureStore( ) 에서 slice들을 하나의 reducer 로 맵핑해줄 수 있다. 
```js
// ./model/index.js

import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./counter-slice";
import authSlice from "./auth-slice";

  

const ReduxStore = configureStore({
  reducer: { counter: counterReducer, auth: authSlice },
});

  
export default ReduxStore;
```
아래와 같이 reducers 객체에 함수를 정의 해주면  기존에 action 과 switch 문으로 함수를 반환했던 방식과 다르게 Redux-toolkit이 함수 이름으로 action을 반환한다.
> 불변 라이브러리가 내장되어 있기 때문에 state.counter++  같은 문법이 성립한다.

``` js
// ./model/counter-slice.js
import { createSlice } from "@reduxjs/toolkit";
const intialCounterState = { counter: 0, showCounter: true };

  
//combine Reducer
const counterSlice = createSlice({
  name: "counter",
  initialState: intialCounterState,
  reducers: {
    add(state) {
      state.counter++;
    },
    sub(state) {
      state.counter--;
    },
    add5(state, action) {
      state.counter = state.counter + action.payload;
    },
    toggle(state) {
      state.showCounter = !state.showCounter;
    },
  },
});

export const counterActions = counterSlice.actions;
export default counterSlice.reducer;
```

Component에서 store을 사용하는 방법이다. redux를 사용하는 방법과 유사하나 , createSlice( ) 사용했기 때문에 payload가 필요한 경우를 제외하면 action 이 필요하지 않다.

```jsx
import { useDispatch, useSelector } from "react-redux";
import { authActions } from "../store/auth-slice";

const Header = () => {
  const dispatch = useDispatch();
  //state.auth.isAuth 에서 auth 는 configureStore에 매핑해준createslice ()의 반환값 이다.
  const isAuth = useSelector((state) => state.auth.isAuth);
  function logoutHandler() {
    dispatch(authActions.logout());
  }
  if (!isAuth) {
    return (
      <header >
        <h1>Redux Auth</h1>
      </header>
    );
  }
  return (
    <header className={classes.header}>
      <h1>Redux Auth</h1>
      <nav>
        <ul>
          <li>
            <a href="/">My Products</a>
          </li>
          <li>
            <a href="/">My Sales</a>
          </li>
          <li>
            <button onClick={logoutHandler}>Logout</button>
          </li>
        </ul>
      </nav>
    </header>
  );
};

export default Header;

```

