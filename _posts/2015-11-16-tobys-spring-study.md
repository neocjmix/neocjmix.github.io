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
 - 예외 발생 테스트 --> `@Test(expected = ExpectedException.class)`
 - test with spring context
 	+ Annotation 설정

		```java
		@RunWith(SpringJUnit4ClassRunner.class)
		@ContextConfiguration(classes = BeanConfiguration.class, loader = AnnotationConfigContextLoader.class)
			//or
		@ContextConfiguration(location = "/applicationContext.xml")
			//or
		@ContextConfiguration
		```
		http://blog.outsider.ne.kr/860
	+ 같은 설정파일이면 여러개의 Context가 만들어지지 않고 하나로 공유된다.
	+ context를 변경할 시 클래스 또는 메소드에 `@DirtiesContext` , `@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)` 를 사용하면 공유하지 않고 다른 테스트에 영향을 끼치지 않도록 새로 만들어준다.

##3장

 - 템플릿/콜백 패턴

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
 	+ SQLException.getSQLState() 는 부정확하다.
 	+ SQLException --> DataAccessException 으로 래핑
 	+ 실제로는 DataAccesssException 를 상속한 다양항 예외를 throw 한다.
 	+ DataAccesssException
 		* DataIntegrityViolationException
 			* DuplicateKeyException - [주의]JDBC에서만 발생
 		* InvalidDataAccessResourceUsageException
 			-  BadSqlGrammarException - JDBC
 			-  HibernateQueryException - HIbernate
 			-  TypeMismatchDataAccesException
 		*  OptimisticLockingFailureException - 낙관적인 락킹 예외 (동시 업데이트 방지)
 			-  ObjectOptimisticLockingFailureException
 				+  HIbernateOptimisticLockingFailureException
 				+  JdpOptimisticLockingFailureException
 		*  IncorrectResultSizeDataAccessException - queryForObject 등에서 결과 갯수
 			-  EmptyReaultDataAccessException
 -  SQLException으로부터 직접 Translation
 	
 	```java
 	@Test(expected = DuplicateKeyExceptionException.class)
	    public void testSqlExceptionTranslate() throws Exception {
	        try{
	            ud.add(user1);
	            ud.add(user1);
	        }catch(DuplicateKeyException e){
	            SQLException sqlException = (SQLException)e.getRootCause();
	            SQLExceptionTranslator sqlExceptionTranslator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
	            DataAccessException dataAccessException = sqlExceptionTranslator.translate(null, null, sqlException);
	            throw dataAccessException;
	        }
	    }
 	```

