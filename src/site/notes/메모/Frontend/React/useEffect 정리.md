---
{"dg-publish":true,"permalink":"//frontend/react/use-effect/","title":"useEffect 정리","tags":["react"]}
---


React 에서 부수 효과란 Jsx 를 랜더링한 이후에 비동기로 처리되어야 하는 동작들을 의미한다.

```js 
//./app/api/route.js
export async function GET() {

  return Response.json({ msg:'hello' }, { status: 200 });

}
```

```js
//./app/route.js
"use client";

import { useState } from "react";

  

export default function Home() {
  const [data, setData] = useState(["test"]);

  fetch("/api")
    .then((response) => response.json())
    .then((data) => setData(data));

  

  return <div>{data}</div>;

}

```

위의 코드의 경우에 처음 div 태그 안이 test 로 랜더링 된다 . 그 이후에 fecth 에 의해 state 가 변경되면 페이지를 다시 랜더링한다.
현재 api 는 고정값을 반환 하기 때문에 state 에 변경이 없어 첫 실행 이후에 값이 변경되지 않는다. 그러나, api 가 아래와 같다면 State의 변화를 감지하기 때문에  계속해서 페이지는 랜더링된다. **무한루프에 빠진다.**

```js

export async function GET() {

  return Response.json({ msg: Math.random() }, { status: 200 });


}
```

## 1. useEffect 
---
부수효과는 위와 같이 무한루프를 야기한다. 이를 문제를 해결하기 위해서 react 에서는 useEffect 훅을 사용한다.

![Pasted image 20240116163011.png](/img/user/0.%20%EC%9D%B4%EB%AF%B8%EC%A7%80/Pasted%20image%2020240116163011.png)

아래와 같이 useEffect 에 의존성배열을 빈 배열로 넣어주면 ComponentDidMount 시점에 실행된다.

```js
useEffect(()=>{
  fetch("/api")
    .then((response) => response.json())
    .then((data) => setData(data));
},[])
```

#### 1.a 사용예제
---
처음 랜더링된 이후에 실행된다는 useEffect의 특징에 이용하면 아래와 같은 코드가 가능하다.
만약, 아래의 코드에 useEffect 를 사용하지 않는다면 jsx 와 useRef가 연결되기전에 코드가 실행되기 때문에 에러가 발생한다.

```js

cons modal=({open})=>{
	const dialog=useRef();

	useEffect(()=>{
	if(open){
		dialog.current.showModal();
	}
	else{
		dialog.current.close();
	}
	},[open])
	
	return (
		<dialog ref={dialog}>
			HI
		</dialog>
		)
}
```


#### 1. b Cleanup 함수
---
componentWillUnmount 에 해당한다.
실행 시점
- 해당 effect 함수가 다시 작동하기 전 update 전
- 해당 component가 사라질때


```js

useEffect(() => {
	const inerval=setInterval(()=>{
		consloe.log('tic toc!')
		setRemaingTime((prev)=>prev-10)
	},10)
	return ()=>{
		clearInterval(interval)
	}
  },[])

```

> ####  의존성 배열에 함수가 들어갈때
js에서 함수는 변수에 정의 되기 때문에 값이 계속 변한다. 따라서 앱 컴포넌트 함수가 실행될 때마다 함수가 재정의된다. 
만약 이를 useEffect 내부에서 실행한다면  무한루프를 야기 한다.


#### useCallback
---
위에서 언급한 useEffect 의존성 배열에 함수 사용시 발생하는 무한루프 현상을 useCallback 훅으로 해결할 수 있다. 
useCallback 은 함수를 메모리에 저장한 후에 app 컴포넌트가 다시 실행될때 저장된 메모리를 다시 사용한다.

```js

const foo= useCallback(()=>{
	console.log('test')

},[])
```

