---
{"dg-publish":true,"permalink":"//next-js/cors/"}
---

next js 에서 Cors 문제 발생시에 rewrites, redirect 를 next.config.js 에 삽입해서 해결할 수 있다. 두 키워드 모두 Cors 문제를 해결 한다는 공통점이 있으나 동작은 살짝 다르다.


#### rewrite
---

```js
/** @type {import('next').NextConfig} */

const nextConfig = {
  async rewrites() {
    return [
      {
        source: "/api/:path*",
        destination: `http://localhost:3000/:path*`,
      },
    ];
  },
};

module.exports = nextConfig;
```

source 의 value 값 "api" 일 경우에  프록시를 통하여 요청한다. 그러나 URL 이 바뀌지 않은 상태로 이동한다.

#### redirect
---

```js
/** @type {import('next').NextConfig} */

const nextConfig = {
  async redirect() {
    return [
      {
        source: "/api/:path*",
        destination: `http://localhost:3000/:path*`,
        permanent: false,
      },
    ];
  },
};

module.exports = nextConfig;
```
다른 점들은 rewite 와 동일하나 해당 Url 로 변경된다. API 의 경로를 감출 수 있다.
- `permanent` : true or false
    - true인 경우 클라이언트와 search 엔진에 redirect를 영구적으로 cache하도록 지시하는 `308` status code를 사용
    - false인 경우 일시적이고 cache되지 않은 `307` status code를 사용