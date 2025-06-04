---
{"dg-publish":true,"permalink":"///python/","title":"python 오프라인 배포","tags":["python"]}
---

#### 1. Requirements.txt 추출

```
conda list -e > requirements.txt
```

#### 2. 패키지 다운로드

```
pip download -r .\requirements.txt
```

#### 3. 패키지 설치 (설치환경에서)


```shell
  * 전체 파일 설치

pip install --no-index --find-links="./" -r requirements.txt

   * 목록 중 개별 패키지 선택 설치

pip install --no-index --find-links="./" 패키지파일명.확장자
```