---
{"dg-publish":true,"permalink":"//frontend/react/key/","title":"key 를 사용해서 컴포넌트를 리랜더링 하기"}
---


```js
import { useState, useEffect } from "react";


function App() {
  const [parentData, setParentData] = useState(0);
  const handleSetParentData = () => {
    setParentData((prev) => prev + 1);
  };
  return (
    <div>
      <button onClick={handleSetParentData}>Click!</button>
      <div> {parentData}</div>
      <Child  initialValue={parentData} />
    </div>
  );
}

export default function Child({ initialValue }) {
  const [data, setData] = useState({ value: initialValue });
  const handleSetData = () => {
    setData((prev) => ({ value: prev.value + 1 }));
  };
return (
    <>
      <button onClick={handleSetData}>Click!</button>
      <div>{data.value}</div>
    </>
  );
}}

```

위의 코드에서 App 컴포넌트의 버튼을 클릭하여도 Child 컴포넌트의 값이 변하지 않는다. 

> Props 값이 변경되어 Compoent 랜더링이 다시 일어나야 하지 않는가?

변경되지 않는 이유는 React에서 같은 위치에 존재하는 동일한 컴포넌트를 Key 로 구분한다. 그러나 위에서는 Key 를 정의 해주지 않았다. 그렇기 때문에  처음 Child 컴포넌트가 initialValue로 state 를 초기화 한 이후에 props 를 변경해주더라도 Key를 사용하지 않았기 때문에 React 는 Ui 트리에서 변경을 감지 하지 못한다.

>React에서는 실제 Dom 에서의 위치를 통해 State가 어떤 컴포넌트에 속하는지 추적한다. 

#### 해결방법 
___
```js
import { useState, useEffect } from "react";


function App() {
  const [parentData, setParentData] = useState(0);
  const handleSetParentData = () => {
    setParentData((prev) => prev + 1);
  };
  return (
    <div>
      <button onClick={handleSetParentData}>Click!</button>
      <div> {parentData}</div>
      <Child key={parentData} initialValue={parentData} />
    </div>
  );
}

export default function Child({ initialValue }) {
  const [data, setData] = useState({ value: initialValue });
  const handleSetData = () => {
    setData((prev) => ({ value: prev.value + 1 }));
  };
return (
    <>
      <button onClick={handleSetData}>Click!</button>
      <div>{data.value}</div>
    </>
  );
}}

```
App 컴포넌트에서 parentData 상태값이 변경 될때 마다 Child Component key 를 변경해준다. 이는  react 가 컴포넌트를 식별할 수 있는 고유한 값이므로 React가 컴포넌트에 변화가 일어 났음을 감지할 수 있다.

혹은, useEffect 를 사용하여 컴포넌트를 props 변경을 감지할 수 있지만
useEffect의 실행 주기는 랜더링 된 이후에 일어나기 때문에 불필요한 랜더링이 추가로 일어난다. 
```js
import { useState, useEffect } from "react";


function App() {
  const [parentData, setParentData] = useState(0);
  const handleSetParentData = () => {
    setParentData((prev) => prev + 1);
  };
  return (
    <div>
      <button onClick={handleSetParentData}>Click!</button>
      <div> {parentData}</div>
      <Child key={parentData} initialValue={parentData} />
    </div>
  );
}

export default function Child({ initialValue }) {
  const [data, setData] = useState({ value: initialValue });
  usEffect(()=>{
	  setData({ value: initialValue })
  },[initialValue])
  const handleSetData = () => {
    setData((prev) => ({ value: prev.value + 1 }));
  };
return (
    <>
      <button onClick={handleSetData}>Click!</button>
      <div>{data.value}</div>
    </>
  );
}}

```