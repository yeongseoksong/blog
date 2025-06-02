---
{"dg-publish":true,"permalink":"//db/mysql-table-space/"}
---

MySQL 8.0 기준으로 총 5가지의 Tablespace가 존재한다.

1. System Tablespace
2. File-Per-Table Tablespace
3. General Tablespace
4. Undo Tablespace
5. Temporary Tablespace

당연히 데이터 파일에 아무런 체계없이 데이터를 무작정 저장하지는 않는다. 

오라클 DBMS를 예로 들면, 오라클은 데이터 파일 내부의 공간을 4개의 논리적인 공간으로 나눠놨다.

1. **테이블스페이스** (Tablespace)  
    C++의 네임스페이스같은 개념이다.  
    아래의 Segment를 담는 컨테이너로서, 여러 개의 데이터 파일을 묶어놓은 논리적인 저장공간을 가리킨다.
2. **세그먼트** (Segment)  
    데이터 저장이 필요한 하나의 객체를 저장하는 단위. ( 테이블, 인덱스 등 )  
    예를 들어, board 테이블을 만들면 하나의 세그먼트가 생성이 되는 것.
3. **익스텐트** (Extent)  
    공간을 확장하는 단위. 예를 들어 board라는 테이블이 있으면 이는 하나의 세그먼트로 할당이 되어 데이터가 저장이될텐데,  
    board 테이블을 최초 생성할 때에는 데이터가 없으니까 익스텐트를 하나만 생성해도 된다. 그러다가 레코드가 점점 늘어나면 저장 공간의 추가 할당이 필요할텐데, 그때의 확장 단위.
4. **데이터 블록** (Data Block)   
    DBMS가 디스크에서 데이터를 읽고 쓰는 단위.  
    디스크 I/O 단위가 블록이므로, 특정 레코드 하나만 읽고 싶어도 해당 블록을 통째로 읽는다.  
    오라클은 디폴트로 8KB 크기의 블록을 사용한다. (설정에 따라 2, 4, 16, 32KB도 사용할 수 있음)  
    그러니까 예를 들어 디스크에서 1Byte만 읽어오고싶어도 어쩔 수 없이 8KB를 읽어와야한다. 최소 단위가 블록이니깐.

Tablespace 공간을 표현하면 아래 그림과 같다.

> MySQL 은 테이블 별로 전용의 테이블 스페이스를 사용한다.

![](https://blog.kakaocdn.net/dn/EfrLq/btqCn0wtdH7/GlnU22P2ftQgDZhjeAjgQ1/img.png)



출처 https://pangtrue.tistory.com/190