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
| 	 			| 초난감 DAO|
|템플릿 메쏘드 패턴	| getConnection 추상 메쏘드 |
|전략패턴			| ConnectionMaker interface 파라미터 DI |
|팩토리			| DaoFactory|


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
