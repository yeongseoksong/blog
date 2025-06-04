---
{"dg-publish":true,"permalink":"//javas-script/array-buffer-typed-array-int8-array-int16-array/","title":"ArrayBuffer,TypedArray( Int8Array,Int16Array...) 메모","tags":["javascript"]}
---


js 에서 blob 생성자를 호출하려면 보통 데이터를 ArrayBuffer 형식으로 변환하여 매개변수로 넣어주어야 한다.그래서  ArrayBuffer 에 대하여 보던 알아보던 중에 Int8Array ,Uint8Array와 같은 키워드들을 보게 되었는데 Pyhon Numpy,openCv 로 이미지 처리를 했던 프로젝트에서 자주 콘솔에 보았던 에러였기에 정리한다.

#### ArrayBuffer
---
C 언어에서 배웠던 배열이다.  JS Array는 기본적으로 객체 타입이며, 연속적인 데이터 저장을 보장하지 않기때문에 해당 함수를 사용한다.
```javascript
// 8바이트의 배열 생성
const buffer = new ArrayBuffer(8); 

console.log(buffer.byteLength);// 8
```

#### TypeArray
---
ArrayBuffer에 있는 데이터는 직접 수정할 수 없다 . ArrayBuffer 를 다루기위해 TypeArray 를 쓰는데 C 언어에서 배열에 자료형을 붙혀주는 것과 동일하다.
할당된 바이트 크기를 자료형에 따라서 알맞은 크기로 잘라서 읽어들여 다룬다.

- Uint8Array = 매 바이트(8비트) 마다 독자적인 정수로 읽어들여 다루겠다는 의미이다
- Uint16Array = 매 2바이트(16비트) 마다 독자적인 정수로 읽어들여 다루겠다는 의미이다.
- Uint32Array = 매 4바이트(32비트) 마다...
- Float64Array = 매 8바이트 (64비트) 마다 float값으로 읽어들여 다루겠다는 의미로, 가능값에 소숫점 이하가 포함될 수 있음을 의미한다.
```javascript
const buffer = new ArrayBuffer(8); 
const int8arr = new Int8Array(buffer); 
const int16arr = new Int16Array(buffer);
```


#### Python의 Numpy,Opencv 에서 Uint8Array 키워드를 본 이유
---
이미지와 같이 연속되고 큰 바이너리 파일을 처리할때 ArrayBuffer 를 쓴다고 앞서 설명했다 . Uint8Array 를 사용 하게되면 8비트마다  버퍼의 갚을 읽어 들이기 때문에 0~255 까지의 값을 표현할 수있다 .
Rgb 는 (255,255,255) 와 같은 형식을 가지기 때문에 Uint8Array로 변환하는 작업이 필요 했던것이다.

> float32 같은 타입도 사용가능한데  부동소수점으로 더 민감한 이미지 처리 작업에 이용될 수 있다.

