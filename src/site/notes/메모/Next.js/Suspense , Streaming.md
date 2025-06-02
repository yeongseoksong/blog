---
{"dg-publish":true,"permalink":"//next-js/suspense-streaming/","title":"Suspense , Streaming","tags":["nextjs"]}
---

#### SSR 절차
---

![[server-rendering-without-streaming-chart.avif]]
1. 먼저, 특정 페이지의 모든 데이터를 서버에서 가져옴
2. 그런 다음 서버는 페이지의 HTML을 렌더링
3. 페이지의 HTML, CSS 및 JavaScript가 클라이언트로 전송
4. Html , Css 로 비대화형 사용자 인터페이스 생성
5. hydration (react 와 4 의 결과물이 상호 작용 가능케함)
   
 1 ~ 5 의 과정이 순차적으로 이루어져 이전 단계가 완료 되기전에는 다음 단계가 차단된다. 즉 ,  hydration 이 되기 까지 4 단계 까지의 작업이 이루어져야 한다.

만약,  페이지의 크기가 매우 크다면 사용자는 페이지와 상호작용 하기까지 오래 걸린다. 이러한 경우에 스트리밍을 사용할 수 있다. (청크로 분할하여 전송)


#### 스트리밍
---

-  loading.tsx 
  
  ```tsx
// /app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```


- Suspense :
  ```tsx
  
    return (
    <>
      <Suspense fallback={<Loading />}>
        <Slider />
      </Suspense>
    </>
  );
```