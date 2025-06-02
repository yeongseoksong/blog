---
{"dg-publish":true,"permalink":"//frontend/react/react/","title":"React 코드 스플리팅, 비동기 로딩","tags":["react"]}
---


리액트 코드가 빌드될때 웹팩이 자바스크립트 안의 불필요한 주석, 경고 메시지 , 공백을 제거하여 파일 크기를 최소화 하며,  리소스의 경로들을 설정해 준다.
이때 , 모든 자바스크립트 파일들과 css 과 하나의 파일로 합쳐진다.  CRA로 프로젝트를 빌드시엔 최소 두개 이상의 자바스크립트 파일로 분리하여 빌드한다.
> CRA 는 웹팩의 SplitChunks 기능이 활성화 되어 있다.
> build 파일의 2로 시작하는 파일들 (node_module,main) 은 자주 바뀌지 않는 코드들로 캐싱의 할 수 있다는 이점이 있다.


#### 코드 스플리팅
---
빌드시에 자바 스크립트를 하면 웹팩의 splitchunks 기능을 활용하여 캐싱 효과만 있다. 그러나 싱글페이지인 리액트는 모든 js 파일들이 main에 저장되는데 이는 첫 페이지를 다운로드 할때 로딩이 지연될 수 있음을 의미한다.

이를 해결하기 위해 , **코드 비동기 로딩** 을 사용해 컴포넌트를 필요한 시점에 불러와 사용한다.

#### React.lazy
---
React lazy 를 사용하지 않으면 스플리틍을 위해서는 import( )함수를 state에 등록해야한다. 그렇게 된다면 매번 state 를 생성해주어야 하기 때문에 React.lazy가 등장했다.


```js
const Split = React.lazy(()=>import('./Split))
```


#### Suspense
---
React.lazy 를 사용하여 스플리팅된 컴포넌트가 로드될때 까지 보여줄 JSX를 선언할 수 있다.
```js
import {Suspense} from 'react'

<Supense fallback={<div>loading ... </div>}
	<Split>
</Suspense>
```
