# MyBatis

### 1. MyBatis란?
- SQL Mapping Framework → 쉽고 심플
- 자바 코드로부터 SQL문을 분리해서 관리
- 매개변수 설정과 쿼리 결과를 읽어오는 코드를 제거
- 작성할 코드가 줄어서 생산성 향상 & 유지 보수 편리

### 2. SqlSessionFactoryBean과 SqlSessionTemplate
- **(Interface)**
	- SqlSesstionFactory : **SqlSession을 생성해서 제공**
	- SqlSession: SQL명령을 수행하는데 필요한 메서드 제공
- **(Interface Implement): 구현체(빈으로 등록해야함)**
	- SqlSessionFactoryBean : SqlSessionFactory를 Spring에서 사용하기 위한 빈
	- SqlSessionTemplate: SQL명령을 수행하는데 필요한 메서드 제공. thread-safe(멀티스레드에 안전하다: 여러 스레드가 동시접근해도 가능)
	- 빈 등록
	```
	//root-context.xml
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
		 <property name="dataSource" ref="dataSource"/>\
		 <property name="mapperLocations" value="classpath:mapper/*Mapper.xml"/>
			 //mapper/*Mapper.xml(sql xml문서): 위치지정
	  </bean>  
	  
	 <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">  
		 <constructor-arg ref="sqlSessionFactory"/>  
	 </bean>
	```

### 3. SqlSession의 주요 메서드
※String statement: sql문이 들어있는 xml의 이름<br>
※Object parameter: values(?,?) 또는 where value= ? 이렇게 "?"에 들어갈 값<br>
※ Method의 타입이 int: 실행될 때 1반환, 실행이 되지 않으면 0
|메서드|설명|
|----|----|
|int insert(String statement)<br>int insert(String statement,Object parameter)|insert문을 실행하고, insert된 행의 갯수를 반환|
|int delete(String sstatement)<br>int delete(String statement,Object parameter)|delete문을 실행하고, delete된 행의 갯수를 반환|
|int update(String statement)<br>int update(String statement, Object parameter)|update문을 실행하고, update된 행의 갯수를 반환|
|T selectOne(String statement)<br>T selectOne(String statement, Object parameter)|**하나의 행**을 반환하는 select에 사용<br>parameter로 SQL에 binding될 값 제공|
|```List<E> selectList(String statement)```<br>```List<E> selectList(String statemet, Object parameter)```|**여러 행**을 반환하는 select에 사용<br>parameter로 SQL에 binding될 값 제공|
|```Map<K,V> selectMap(String statement, String KeyCol)``` <br>```Map<K,V> selectMap(String statement, String KeyCol, Object parameter```)|**여러 행**을 반환하는 select에 사용<br>keyCol에 Map의 Key로 사용할 컬럼 지정|

### 4. ```<typeAliases>```으로 이름 짧게 하기 - https://mybatis.org/mybatis-3/configuration.html#typeAliases

- [src/main/resources/mybatis-config.xml]
	```
	<typeAliases>
		<typeAliases alias="TestDao" type="com.repository.project1.domain.TestDao"/>
	</typeAliases>	
	```
	※ alias는 대소문자 구분 안함<br>
	
# MyBatis로 DAO작성하기

### 1. BoardDao의 작성
1. DB테이블 생성
2. MapperXML & DTO작성
3. DAO인터페이스 작성
4. DAO인터페이스 구현&테스트

### 2. DTO란? (Data Transfer Object)
계층간의 데이터를 주고 받기 위해 사용되는 객체

※흐름<br>
1. 클라이언트 요청 
2. @Controller (DTO에 요청한 데이터를 담아서 Service로)
	- 요청과 응답을 처리
	- 데이터 유효성 검증
	- 실행 흐름을 제어(Redirect&forward) 
3. @Service (DTO를 Repository로 전달)
	- 비지니스 로직 담당
	- 트랜잭션 처리
4. @Repository (DTO에 담은 데이터를 DB에 저장)
	- 순수 Data Access 기능(DAO)
	- 조회, 등록, 수정, 삭제
5. 조회: 조회한 데이터를 DTO에 담음
6. 각 계층(@Repository > @Service > @Controller)을 거쳐서 클라이언트에 응답
