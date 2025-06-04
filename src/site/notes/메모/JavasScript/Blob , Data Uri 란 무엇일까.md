---
{"dg-publish":true,"permalink":"//javas-script/blob-data-uri/","title":"Blob , Data Uri 란 무엇일까","tags":["javascript"]}
---


#### Data Uri 
---
데이터 URL data: 스킴이 접두어로 붙은 URL 로 컨텐츠 작성자가 작은 파일을 문서내에 인라인으로 삽입할 수 있도록 해준다.

아래와 같은 구조를 갖는다.

```
data:[<mediatype>][;base64],<data>
```
 - data : 접두어
 - mediatype : mime 타입
 - base64 : 데이터가 텍스트가 아닌경우에  인코딩 방식
 - data : data 자


예를 들면 아래와 같다. 
``` javascript
const data_uri_sample_1 = 'data:text/plain;base64,SGVsbG8sIFdvcmxkIQ%3D%3D';

const data_uri_sample_2 = 'data:text/html,%3Cscript%3Ealert%28%27hi%27%29%3B%3C%2Fscript%3E'
 
```

---
>#### 자바스크립트에서  Base 64 를 다루는 함수
>```javascript
const encodedData = btoa("Hello, world"); // 문자열 인코딩
const decodedData = atob(encodedData); // 문자열 디코딩




#### Blob **(Binary Large Object):**
----
Blob은 이진 데이터의 집합을 하나의 엔티티로 저장하는 객체이다.
대용량의 이진 데이터를 다룰 때 유용하며, 파일 다운로드나 이미지 업로드와 같은 작업에 많이 사용된다.

아래는 자바스크립트에서 Blob 인터페이스를 정의한 방식이다.

![1_GPasxj7xFaL72Y4wgEr_8w.webp](/img/user/images/1_GPasxj7xFaL72Y4wgEr_8w.webp)

생성자 함수에서 BlobParts 는 blob 객체 , UsvString , ArrayBuffer를 인수로 받을 수 있다.
Blob 객체의 데이터를 다루는 함수들이 존재한다.

[참고]  [[ArrayBuffer,TypedArray( Int8Array,Int16Array...) 메모]]
 
아래는 자바스크립트에서 블롭 객체를 생성하는 명령어이다.

```javascript
const blob = new Blob([arrayBuffer], { type: 'image/png' });

```

![Pasted image 20231206145502.jpg](/img/user/images/Pasted-image-20231206145502.jpg)


> #### File 객체
> Blob 객체를 상속하는 객체이다 .  name,lastModified 속성을 추가로 갖는다.![Pasted image 20231206145753.jpg](/img/user/images/Pasted-image-20231206145753.jpg)
> 


#### createObjectUrl
---
Blob 객체를 나타내는 URL을 포함한  DOMString 을 생성한다 (blob:url)
Blob Url 은 생성된 브라우저에서만 유효하기 때문에 (file:url) 보다 보안상 이점이 있다.

```javascript
const blobUrl = URL.createObjectURL(blob);
// GC가 동작 하지 않기 때문에 반드시 할당 해제해주어야한다.
URL.revokeObjectURL(blobUrl)
```


