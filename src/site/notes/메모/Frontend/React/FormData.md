---
{"dg-publish":true,"permalink":"//frontend/react/form-data/"}
---

#### FormData
---
폼을 쉽게 보내도록 도와주는 객체이다. Form 태그 안에 존재하는 요소들의 필드 전체를 가져올 수 있다.
아래의 예시에서는 data 변수에 두개의 input 요소를 가져올 수 있다. name 속성이 필수 적이다.
- http 통신에서 FormData 객체를 바디로 받을 수 있다.
- Content-Type 속성은 multipart/form-data
- 
```js
const App=()=>{
	const handleSubmit=(event)=>{
		const formdata=new FormData(event.target);
		 // `[key, value]` 쌍의 배열을 반환
        const data=Object.fromEntries(formdata.entries())
        console.log(data) // {email: "---@---.com", password="password"}
	}
	return(
		<form onSubmit={handleSubmit}>
			<input
				type="email"
				name="email"
			/>
				<input
				type="password"
				name="password"
			/>
		</form>
	)
}
```

fromEntries() 를 사용해서 위와 같이 FormData 의 전체 key,value 를 가져올 수 도 있고,
FormData.get() , FormData.append ...  의 함수로 FormData에 접근할 수 있다.

만약 아래와 같이 동일한 name 을 가지는 요소가 여러개 일때는 getAll () 함수를 사용하면 배열을 반환한다.
```js
const App=()=>{
	const handleSubmit=(event)=>{
		const formdata=new FormData(event.target);
        console.log(acquisition=fd.getAll('checkBox')) // ['google','naver']
	}
	return(
		<form onSubmit={handleSubmit}>
		
		   <input
              type="checkbox"
              name="checkBox"
              value="google"
            />
            <label >Google</label>
               <input
              type="checkbox"
              name="checkBox"
              value="naver"
            />
            <label >Google</label>
		</form>
	)
}
```


#### Validation
---
useState , useRef 를 사용 할 경우 hanlder 함수에 검증 코드를 추가할 수 있지만 코드가 복잡해진다. 따라서 브라우저에서 제공하는 기능을 사용한다.

- required
- minlength, maxlength
- min,max
- pattern :   pattern="010-\d{4}-\d{4}$"
