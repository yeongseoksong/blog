---
{"dg-publish":true,"permalink":"//frontend/react/module-css-css/","title":"Css.module.css 로 css 스코핑","tags":["react"]}
---

1.  기존의 headr.css 파일을 header.module.css로 변경
2.  자바스크립트 객체로 구분되기 떄문에 아래와 같이 import
```javascript
//import './header.css'
import style from './header.module.css'


const hello =()=>{
return (
	<p style.hello>Hello </p>
)
}
```

#### 장점
- 레거시 프로젝트에 리액트를 도입할 때 
- **CSS 클래스 네이밍 규칙을 만들기 귀찮을 때**