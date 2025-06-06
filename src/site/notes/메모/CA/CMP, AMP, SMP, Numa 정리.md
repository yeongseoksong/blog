---
{"dg-publish":true,"permalink":"//ca/cmp-amp-smp-numa/"}
---



#### CMP
- 칩 멀티 프로세서
- 멀티코어 cpu를 의미한다.
- **운영체제 입장에**서는 코어는 개별적인 논리적 CPU로 보인다.
- CMT (chip multi threading ):  메모리 스톨 현상을 위해 하드웨어 스레드를 둔다.


#### AMP
- 비대칭 다중 처리기
- 마스터 처리기가 모든 스케줄링 결정과 I/O 처리
- 마스터 서버가 전체 시스템 성능을 저하할 수 있다.
#### SMP
- 대칭 멀티 프로세서
- 한 시스템에 여러개의 프로세서가 존재
- **스케줄링 전략**
	- SQMS : 공통 대기큐 방식
	- MQMS : 프로세서별 스레드 큐 방식
		- load imbalance 문제가 있다.

#### NUMA ( non- uniformed memory access )
- smp 구조에서 메모리 병목을 개선하기 위한 모델
- **시스템 전체적으로는 메모리 주소를 공유하지만, 물리적인 메모리 위치는 떨어져 있을 수 있다.**
- 한 프로세서와 가까이 있는 메모리로의 접근이 다른 메모리보다 빠르다. 메모리 접근 속도가 물리적인 프로세서와 메모리의 위치에 따라 결정되기에 **'비균일' 메모리 접근 속도**가 만들어지는 것이다. 따라서 **NUMA 환경이라면 이것을 인지해 최적화해야 할 것**이다.