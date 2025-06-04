---
{"dg-publish":true,"permalink":"//frontend/css/font-size/","title":"반응형 웹을 위한 font-size 초기화","tags":["css"]}
---

em, rem 개념을 이해한 후에 단순히 반응형 웹을 제작하기 위해서는 아래와 같이 브라우저 크기별로 font-size를 임의로 지정해주면 되는 줄 알았다. 
[[메모/Frontend/Css/em 과 rem 이란\|em 과 rem 이란]]
```css
@media (max-width: 767px) {
    html {
        font-size: 10px;
    }
}

  

@media (min-width: 768px) and (max-width: 991px) {
    html {
        font-size: 15px;
    }
}


@media (min-width: 992px) and (max-width: 1199px) {
    html {
        font-size: 20px;
    }
}

  

@media (min-width: 1200px) {
    html {
        font-size: 25px;
    }
}
```

이렇게 프로젝트에 적용한 이후 브라우저 크기를 조절 했을때 브라우저 넓이에 따라 정상적으로 콘텐츠들의 크기가 줄어든다.

> 다른 css 요소들의 단위가 rem 이여야 한다.
> 

#### 반응형 폰트 공식
---

하지만, 경계값 근처에서 웹 페이지가  부드럽게 전환되지 못해서 더 좋은 방법을 찾았다 아래와 같다.

```css
html {
    font-size: calc(15px + 0.390625vw);
}
```

이 코드를 적용한다면 모든 경계 값 에서 완벽한 폰트 크기를 얻을 수 있다.

| 화면크기 | 폰트 크기 |
| -------- | --------- |
| 320px    | 16px      |
| 768px    | 18px      |
| 1024px   | 19px      |
| 1280px   | 20px      |
| 1536px   | 21px      |
| 1920px   | 23px      |
| 2560px   | 25px      |

#### 반응형 패딩 공식
```css
calc(8px + 1.5625vw)
```
