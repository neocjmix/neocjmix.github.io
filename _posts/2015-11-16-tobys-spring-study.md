---
layout: post
title: Toby's Spring Study
categories: []
tags: []
published: True

---

##1장

|   |   |
|---|---|
| . 			| 초난감 DAO|
|템플릿 메쏘드 패턴	| getConnection 추상 메쏘드 |
|전략패턴		| ConnectionMaker interface 파라미터 DI |
|팩토리			| DaoFactory|
|스프링 빈		| @Bean|


###memo

 - mysql, jdbc 복슴
 	+ resultset, connection, preparedstatement 모두 close 해야 한다.
 	+ primary key, unique, index 관계...
 	+ 내용이 긴 필드는 1:1 테이블 분리 필수
 	+ `PreparedStatement ps = con.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);` -> resultset
 	+ foreign key `pid INT, FOREIGN KEY (pid) REFERENCES PERSON (pid)`
 	+ ALTER TABLE TEXT ADD FOREIGN KEY (mid) REFERENCES MESSAGE(mid);
 	+ last_insert_id()
 	+ transaction with mysql
 	
	 	```sql
	 	START TRANSACTION;
			INSERT INTO MESSAGE (pid) VALUES (33);
			INSERT INTO TEXT (tid, content) VALUES (last_insert_id(),"asdf");
		COMMIT;
	 	```
 	
 	+ transaction with jdbc
	 	
	 	```java
	 		try{
	 			con.setAutoCommit(false);
	 			//...do queries
				con.commit();
	 		}catch(SQLException e){
				con.rollback();
	 		}finally{
	 			con.setAutoCommit(true);
	 		}
		```

##2장
예외 발생 테스트 --> `@Test(expected = ExpectedException.class)`

##3장

|   |   |
|---|---|
|템플릿/콜백 패턴	| JdbcTemplate |

##4장

###예외
 - 에러
 	+ 시스템 에러. 트라이캐치로 처리 불가능
 - 예외
 	+ 언체크예외 == 런타임예외 extends RuntimeException
 		* 안잡아도 됌
 		* 코드 잘못
 		* NullPointerException
 		* IllegalArgumentException
 		* 계속 throw 할 필요 없이 잡고 싶을때만 잡으면 됌
 	+ 체크예외 : 일반적인 예외
 		* 주로 발생할 수 있는 외부의 예외상황
 		* 상황을 복구할 가능성이 있음
 		* 꼭 처리해야함
 		*  try ... catch 또는 throw 안하면 컴파일에러
 		*  최근 서버환경의 프로그램에서는 활용도가 떨어
 			-  대응이 불가능하다면 빨리 런타임 예외로 전환
 - 스프링
 	+ SQLException.
 	+ SQLException --> DataAccessException 으로 래핑
 	+ 실제로는 DataAccesssException 를 상속한 다양항 예외를 throw 한다.
 		* DuplicateKeyException
