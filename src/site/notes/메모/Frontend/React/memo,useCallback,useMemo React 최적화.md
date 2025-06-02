---
{"dg-publish":true,"permalink":"//frontend/react/memo-use-callback-use-memo-react/","title":"useCallback,useMemo React 최적화","tags":["react"]}
---

#### 결론
---
**useMemo : 계산 결과를 저장**
**useCallback : 자식 컴포넌트에 Callback 함수를 전달하기 위해 사용**
**Memo : 자식 컴포넌트에서 Callback 함수 변경을 감지**


#### memo
---
old prop value , new prop value 를 비교해 변경이 있어야 Component가 실행된다.
```js
const Counter = memo((props)=>{
	return ( <div>render!<div>>)
})
```

- memo() 함수는 실행되기전에 props의 변경점을 확인해야 하기 때문에 비용이 발생한다.

#### useMemo
---
```js
const memoizedValue=useMemo((
	return memo*memo
)=>[memo])
```

memo 함수와 비슷하게 의존성 배열에 추가된 요소가 변경되기전에  동작하지 않는다.  의존성 배열 요소중에 하나라도 변경이 감지되면 callback 함수를 실행한다. **그렇지 않는다면 이전 결과 값을 가지고 있다.**
#### useCallback
---
useCallback 함수 또한 위와 같다. 단 , Component가 다시 랜더링 될 때마다 함수의 메모리 주소값이 변경되는 것을 방지 해준다.
즉, 의존성 배열이 변경되기전에는 이전에 생성했던 함수를 재사용한다.

#### ex-1 )

```javascript
import React, { useState, useEffect } from "react";

function Profile({ id }) {
  const [data, setData] = useState(null);

  const fetchData = useCallback(
    () =>
      fetch(`https://test-api.com/data/${id}`)
        .then((response) => response.json())
        .then(({ data }) => data),
    [id]
  );

  useEffect(() => {
    fetchData().then((data) => setData(data));
  }, [fetchData]);


}
```

props.id 값이 변경될때마다 fetchData 함수가 변경되고 이를 useEffect가 감지해 함수를 실행한다. 처음 랜더링 되었을때 , id 값이 변경되었을때에만 실행된다.


#### ex-2)

```jsx
const Light=Memo(function Light({ room, on, toggle }) {
  console.log({ room, on });
  return (
    <button onClick={toggle}>
      {room} {on ? "💡" : "⬛"}
    </button>
  );
})
```

Memo ( ) 로 컴포넌트를 감싸주었기 때문에 Light 컴포넌트는 props 의 값이 변경 될때 리랜더링 된다. 여기서 toggle 값은 부모 컴포넌트에서 전달되는 함수 이다. 만약 , 이 함수를 어떠한 조치 없이 전달해 준다면 부모컴포넌트가 랜더링 될때마다 함수는 다시 생성되어 Light 함수에 전달될 것이다.
**Memo() 함수를 썼을때 props 중 함수가 존재한다면 유의 해야 한다.**

```jsx
import React, { useState, useCallback } from "react";

function SmartHome() {
  const [masterOn, setMasterOn] = useState(false);
  const [kitchenOn, setKitchenOn] = useState(false);

const toggleMaster = useCallback(() => setMasterOn(!masterOn), [masterOn]);
  const toggleKitchen = useCallback(
    () => setKitchenOn(!kitchenOn),
    [kitchenOn]
  );


  return (
    <>
      <Light room="침실" on={masterOn} toggle={toggleMaster} />
      <Light room="주방" on={kitchenOn} toggle={toggleKitchen} />
    </>
  );
}
```
이 문제를 useCallback 함수를 사용하여 해결할 수 있다. 위와 같이 부모 컴포넌트에서 useCallback 으로 handler 함수를 감싸준 후에 전달하게 되면
각 state 값이 변할때만 해당 Light Component에 랜더링이 동작하게 된다.


