---
{"dg-publish":true,"permalink":"//frontend/react/state/","title":"State 객체 변경 편하게 하기","tags":["react"]}
---

Spread 문법을 사용해서 불변성을 유지하며 State 를 변경할 수 있다.

```javascript
   const [curState,setCurState]=useState({
        a:10000,
        b:1200,
        c:6,
        d:10
    })

    const handleChange=(stateType,newValue)=>
    {
        setUserInput(pre=>{
            return{
                ...pre,
                [stateType]:newValue
            }
        });
    }
```

