---
{"dg-publish":true,"permalink":"//java/intellij-heap-dump/"}
---



**Run > Edit Configurations > modify options > Add Vm options** 

**매개변수**
```
-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
```


- Xms : 힙 최소 크기
- Xmx : 힙 최대 크기
- XX:+HeapDumpOnOutOfMemoryError : OOM이 발생할 경우에 힙덤프 파일을 생성을 한다
    
- XX:HeapDumpPath : 힙덤프가 생성되는 폴더 경로를 지정한다
    
- XX:OnOutOfMemoryError : OOM이 발생할 경우, 수행할 스크립트를 지정한다(보통은 OOM이 발생하면 애플리케이션이 다운되기 때문에 재시작 스크립트를 다시 수행하기도 한다)