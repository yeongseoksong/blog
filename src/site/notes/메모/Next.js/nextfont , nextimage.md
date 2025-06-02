---
{"dg-publish":true,"permalink":"//next-js/nextfont-nextimage/","title":"font , image 최적화","tags":["nextjs"]}
---


#### font 최적화
---
글꼴을 사용할 경우에 브라우저가 처음에 대체 글꼴이나 시스템 글꼴로 텍스트를 랜더링한 이후에 다시 사용자 지정 폰트로 교체해야하기 떄문에 레이아웃 변경이 발생한다. ( 폰트 변경에 의해 )
Next js 는 빌드시에 글꼴 파일을 다운로드하고 정적 파일들과 함께 호스팅한다.
아래는 "next/font" 를 적용하고 tailwind에 선언해주는 방법이다. sans 가 기본 폰트이므로 이를 교체해준다.
```tsx
import { Noto_Sans_KR } from "next/font/google";

const notoSansKr = Noto_Sans_KR({
  subsets: ["latin"], 
  weight: ["100", "400", "700", "900"],
  variable: "--font-noto",
});

return (
		<div className="font-sans">
			<Content>
		</div>
)
```

```tsx
// tailwind.config.ts
const config: Config = {

  content: [
    "./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ["var(--font-noto)"],
      },
    },
  },
  plugins: [],
};
export default config;
```

#### image 최적화
---
기본 img 태그를 사용하면 아래의 문제가 발생할 수 있다.
- 이미지 화면 크기 반응
- 이미지가 로드 시에 레이아웃 변경


"Next/img " 는 이러한 문제를 자동으로 최적화한다.
- 레이아웃 이동 방지
- 이미지 크기 조정
- 이미지 지연 로딩
- Webp 제공
```tsx
import Image from 'next/image' 


const MyImage = props => { 
return (
	<Image 
		src='profile.webp' 
		width={300} height={300} 
		alt='User profile' 
		quality={80} 
	
		layout='fill'// 부모의 넓이와 높이를 따름
		objectFit='cover'// 사이즈를 부모에 맞춰 꽉 채움
		
/>
```
) }