---
{"dg-publish":true,"permalink":"//frontend/react/navigate-props/","title":"navigate 컴포넌트로 props 전달"}
---

```javascript
// home.js
import { useNavigate } from 'react-router';

const handleClick = (e) => {
    const navigate = useNavigate();
    navigate('/edit', { state: e.target.value });
}
```

```javascript
// edit.js
import { useLocation } from "react-router";

const Edit = () => {
    const { state } = useLocation();
    //const state= useLocation.state
    console.log(state);
}
```