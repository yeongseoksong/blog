---
{"dg-publish":true,"permalink":"//os/tlb-hit-cache-hit-memory-hit/"}
---

![cache,tlb,page.png](/img/user/0.%20%EC%9D%B4%EB%AF%B8%EC%A7%80/cache,tlb,page.png)

**​**﻿**​(0)번** : TLB -> hit,  Cache -> hit,  Virtual Memory -> hit

  

 Best한 경우이다. 먼저, TLB가 hit이므로 페이지 테이블을 볼 필요 없다. 즉, 메인 메모리 접근 필요없음. 그리고, cache hit이므로 TLB에 의해 가상 주소 -> 실제 주소, 이 실제주소를 가지고 cache에 접근해 페이지를 접근함.

  

**(1)번 :** ﻿​TLB -> hit, Cache -> miss, Virtual Memory -> hit

  

이 경우는 TLB가 hit이므로 page table에 접근하지는 않는다. 그러나, cache가 miss이므로 페이지를 읽기 위해 메모리 접근 1회가 필요하다.

  

**(2)번 :** ﻿​TLB -> miss, Cache -> hit, Virtual Memory -> hit

﻿​ ​﻿이 경우는 TLB가 miss이므로 page table에 접근해 가상 주소 -> 실제 주소 변환 작업이 필요하다. (메모리 접근 1회) 만약, page table을 접근했는데 valid bit이 0 이라면 page fault가 발생한다.

cache는 hit이므로 더 이상의 메모리 접근은 없다.

  

**(3)번 :** ​﻿TLB -> miss, Cache -> miss, Virtual Memory -> hit

  

이 경우는 TLB가 miss이므로 page table에 접근한다. 또한, cache도 miss 이므로 메모리에 접근해 페이지를 가져온다. 즉, 메모리 접근 2번이다.

  

**(4)번 :** ​﻿TLB -> miss, Cache -> miss, Virtual Memory -> miss

  

이 경우는 최악의 경우로 메모리에서 miss가 발생했으므로 page fault가 발생한다. 즉, 가장 페이지의 Valid bit가 0이므로 운영체제가 제어를 넘겨받게 된다.(디스크로부터 페이지를 가져옴 - 시간이 오래걸려 성능이 저하된다. )

  

**(5), (6), (7)번 :** ​﻿불가능한 경우이다.

  

가상 메모리와 캐시 시스템은 계층구조를 이루며 같이 동작한다. 따라서, 데이터가 메인 메모리에 없다면 그 데이터는 캐시에 있을 수 없다.(디스크 -> 메모리 -> 캐시 )