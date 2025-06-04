---
{"dg-publish":true,"permalink":"//javas-script/gec-fec-this/","title":"스코프체인 , 변수 객체(Gec,Fec) ,this 란","tags":["javascript"]}
---

실행 컨텍스트는 물리적으로 객체의 형태를 가지며, 3가지 프로퍼티를 갖고 있다.

1. 변수 객체 (variable object)
2. 스코프 체인 (scope chain)
3. this value

#### 1. 변수 객체 (VO)
---

실행 컨텍스트가 생성되면, 자바스크립트 엔진은 이 실행 컨텍스트에 필요한 정보를 담을 객체를 생성하는데 이를 **변수 객체**라고 한다.

>  변수 객체는 코드가 실행될 때, 엔진이 참조한다.

####1.a. **글로벌 실행 컨텍스트(Global Execution Context)** 
---
![Pasted image 20231201145629.png](/images/Pasted%20image%2020231201145629.png)
- **전역 객체**(Global Object / GO)를 가리킨다.
- 변수 객체는 _유일_하고, _최상위_에 위치한다.
- 모든 **전역 변수**와, **전역 함수**를 프로퍼티로 갖는다.

> 무조건 하나의 GEC 만 존재하며 어플리케이션이 종료될때까지 유지한다.


#### 1.b. 함수 실행 컨텍스트 (Functional Execution Context)
---
![Pasted image 20231201145550.png](/images/Pasted%20image%2020231201145550.png)
- **활성 객체**(Activation Object / AO)를 가리킨다.
- **지역 변수**와 **내부 함수**를 프로퍼티로 갖는다.
- 프로퍼티에 인수들의 정보를 배열로 담고 있는 객체인 **arguments object**가 추가된다.

>GEC 가 생성된 후에 함수 호출시마다 새로울 실행 컨텍스트가 작성된다 .

#### 2. 스코프 체인 (Scope Chain)
---
![Pasted image 20231201144304.jpg](/images/Pasted%20image%2020231201144304.jpg)
스코프 체인(Scope Chain)은 일종의 리스트로서 전역 객체와 중첩된 함수의 스코프의 레퍼런스를 차례로 저장하고, 의미 그대로 각각의 **스코프가 어떻게 연결(chain)되고 있는지 보여주는 것**을 말한다.
#### 스코프 체인 동작 방식
---
```javascript
var v = "전역 변수";  //Gec
function a() { //function a Execution Context(EC) 
	var v = "지역 변수"; 
	function b() { //function b Execution Context 
		console.log(v); 
	} 
	b(); 
	} 
//Global Execution Context a();
a()  // '지역 변수'
```

![Pasted image 20231201144644.jpg](/images/Pasted%20image%2020231201144644.jpg)

> 함수 실행 컨텍스트가 가리키는  활성 객체에서 먼저 찾아본 후에 
> 스코프 체인이 연결되어 있는 다음 AO (Activation object)혹은  Go(global object) 로 이동하여 변수를 파악한다.
> 존재 하지 마지막은 GO 에도 변수가 존재하지 않을 경우에 
> Reference Error 를 반환한다.

#### 3. This
---
[참고] https://poiemaweb.com/js-this

