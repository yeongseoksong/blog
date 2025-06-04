---
{"dg-publish":true,"permalink":"//frontend/react/reat/","title":"Reat 앱 디버깅"}
---


#### StrictMode
---
대표적인 기능은 감싸져 있는 모든 컴포넌트들을 두번 실행한다.
두번씩 컴포넌트들이 두번씩 증가되면 문제의 존재를 파악하기 더 쉽다. StrictMode 사용시 검사하는 요소는 아래와 같다.
1. **안전하지 않은 생명주기를 사용하는 컴포넌트 발견**
2.  **레거시 문자열 ref 사용에 대한 경고**
3. **권장되지 않는 findDOMNode 사용에 대한 경고**
4. **예상치 못한 부작용 검사**
5. **레거시 context API 검사**
6. **재사용 가능한 상태 보장**

[Document](https://ko.legacy.reactjs.org/docs/strict-mode.html)
또, App 컴포넌트 전체를 감싸는게 아닌 컴포넌트를 일부만 감싸서 검사를 수행할 수 있다.

```javascript
return(
	<>
		<Header/>
//componet1,2 의 자손까지 검사가 이루어진다.
		<StrictMode>
			<div>
				<Component1/> 
				<Component2/>
			</div>
		</StrictMode>
		<Footer/>
	</>

)
```


#### 개발자 도구 디버깅
---
React 또한 브레이크 포인트를 사용해서 디버깅을 할 수있다.
**F12 > 소스 >의심되는 코드 클릭 > APP 조작**
시에 브레이크 포인트에서 

F8: 다음 브레이크 포인트로 이동
**F10 (Step Over):** 현재 실행중인 함수 내부로 진입 하지 않고,다음 라인으로 이동
**F11 (Step Into):** 함수 내부로 진입하여 디버깅

