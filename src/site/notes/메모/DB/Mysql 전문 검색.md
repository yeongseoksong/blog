---
{"dg-publish":true,"permalink":"//db/mysql/"}
---



[[책/real mysql 1/8. 인덱스#8.5.2 전문 검색 인덱스의 가용성\|8. 인덱스#8.5.2 전문 검색 인덱스의 가용성]] 실습 및 테스트 과정
##### 테이블 생성

```sql
create table tb_test(
	doc_id INT auto_increment,
	doc_body TEXT,
	PRIMARY KEY(doc_id),
	FULLTEXT KEY fx_docbody(doc_body) WITH PARSER ngram
) Engine=innoDB;
```

#### 테스트 데이터 삽입

```sql
INSERT INTO tb_test ( doc_body) VALUES 
('애플은 혁신적인 스마트폰 제조사입니다.'), 
( '애플 제품은 디자인이 매우 훌륭합니다.'), 
( '애플의 새로운 아이폰이 출시되었습니다.'), 
( '애플 워치 시리즈는 건강 관리에 좋습니다.'), 
( '애플 생태계는 통합성이 뛰어납니다.'), 
( '애플은 혁신을 지속적으로 추구합니다.'),
( '애플의 맥북은 고성능 노트북입니다.'), 
( '애플 스토어에서 다양한 제품을 구매할 수 있습니다.'),
( '애플의 소프트웨어는 사용자 친화적입니다.'), 
( '애플은 고급 시장을 겨냥한 브랜드입니다.'); 


INSERT INTO tb_test (doc_body) VALUES 
( '삼성은 전 세계적으로 유명한 브랜드입니다.'), 
( '구글은 검색 엔진 시장의 선두주자입니다.'), 
( '마이크로소프트는 다양한 소프트웨어를 제공합니다.'), 
( '소니는 뛰어난 품질의 전자 제품을 만듭니다.'), 
( '테슬라는 전기차 시장을 선도하고 있습니다.'), 
( '페이스북은 소셜 네트워크 플랫폼을 운영합니다.'), 
( '아마존은 온라인 쇼핑의 거대 기업입니다.'),
( '넷플릭스는 인기 있는 스트리밍 서비스입니다.'), 
( '디즈니는 가족 엔터테인먼트의 선두주자입니다.'), 
( '나이키는 스포츠 용품 시장에서 강력한 브랜드입니다.');


INSERT INTO tb_test (doc_body) select doc_body from tb_test
```

#### 불용어 커스텀 하기
```sql
CREATE TABLE test.user_stopword ( word VARCHAR(255) PRIMARY KEY ); 


//불용어 추가
INSERT INTO test.user_stopword (word) VALUES ('애플');


```

#### 불용어 테이블 지정 
```sql
SET GLOBAL innodb_ft_server_stopword_table = 'test/user_stopword';

```


#### 전문 검색 인덱스 재설정 및 재시작
```sql
ALTER TABLE tb_test DROP INDEX fx_docbody;
ALTER TABLE tb_test ADD FULLTEXT INDEX fx_docbody (doc_body) with parser ngram;
```


#### 결과 확인
- n-gram 알고리즘에서 `doc_body` 를  토큰으로 나눈 후 불용어 포함하거나 같은 경우 제외한다. 그 이후 남은 토큰들을 인덱스로 만들어 사용한다.

```sql
SELECT * FROM tb_test WHERE MATCH(doc_body) AGAINST('애플' IN BOOLEAN MODE);
//결과 없음
SELECT * FROM tb_test WHERE MATCH(doc_body) AGAINST('삼성' IN BOOLEAN MODE);
// 두번째 쿼리는 삼성이 포함된 로우들이 출력된다.

```

#### "삼성전자" 라는 데이터를 넣고 출력되는지 확인

- like 삼성% 처럼 잘 동작 하는지 확인한다.
```sql

insert into tb_test (doc_body) values
('삼성전자 테스트 데이터'),
('삼성전자 테스트 데이터1'),
('삼성전자 테스트 데이터2')


select * from tb_test where match(doc_body) against('삼성' in boolean mode) order by doc_id desc;

```

![full text index test img.png](/images/full%20text%20index%20test%20img.png)

#### "애플테스트", "애1플" 데이터 삽입후 확인

```sql
insert into tb_test (doc_body) values
('애플테스트 데이터'),
('애플테스트 데이터1'),
('애테스트 데이터2'),
('애1플테스트 데이터'),
('애1플테스트 데이터1'),
('애1테스트 데이터2')


SELECT * FROM tb_test WHERE MATCH(doc_body) AGAINST('애플' IN BOOLEAN MODE);
// 결과 없음
SELECT * FROM tb_test WHERE MATCH(doc_body) AGAINST('애1플' IN BOOLEAN MODE);
// 추가한 3건의 데이터 출력
```

- 결과를 통해 n-gram "애플테스트" 는 "애플","플테"... "스트" 로 토큰이 불리되어 "애플이" 불용어에 필터링되어 삭제됨을 확인
- "애1플" 의 경우 "애1","1플" 로 토큰이 저장되어 불용어에서 필터링하지 못했다.