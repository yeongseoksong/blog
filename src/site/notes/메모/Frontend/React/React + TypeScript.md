---
{"dg-publish":true,"permalink":"//frontend/react/react-type-script/"}
---


```typescript 
//./models/TodoModel.ts
export default class TodoModel {
  id: string;
  text: string;


  constructor(todoText: string) {
    this.text = todoText;
    this.id = new Date().toISOString();
  }
}
```

```ts
import React from "react";
import Todo from "./Todo";

function App() {
  const todos = [new TodoModel("learn ts"), new TodoModel("Learn TypeScript")];
  return (
    <div>
      <h1>My Todo List</h1>
      <Test items={todos}/>
    </div>
  );
}

export default App;

```

#### 컴포넌트
---
-  화살표 함수
```ts
// ./coomponents/Todo.tsx
import TodoModel from "../models/TodoModel";

const Todo: React.FC<{ items?: TodoModel[] }> = (props) => {
  return (
    <ul>
      {props.items?.map((item) => (
        <li key={item.id}> {item.text}</li>
      ))}
    </ul>
  );
};

  
export default Test;

```
-  function 사용 (권장)
```ts
import TodoModel from "../models/TodoModel";

export default function Todo({ items }: { items?: TodoModel[] }) {
  console.log(items);
  return (
    <ul>
      {items?.map((item) => (
        <li key={item.id}> {item.text}</li>
      ))}
    </ul>
  );
}
// default 값 설정 가능
Todo.defaultProps = {
  items: [new TodoModel("hi"), new TodoModel("Learn TypeScript")],
};


```


#### children 전달
---
```ts
import { ReactNode } from "react";
import TodoModel from "../models/TodoModel";

interface TodoProps {
  items: TodoModel[];
  children?: ReactNode;
}

export default function Todo({ items, children }: TodoProps) {

  console.log(items);
  return (
    <>
      {children}
      <ul>
        {items?.map((item) => (
          <li key={item.id}> {item.text}</li>
        ))}
      </ul>
    </>
  );

}

```

