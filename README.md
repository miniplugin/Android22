## 서버프로그램 구현 (v9.0.0)
 
---

- [학습목차](https://github.com/miniplugin/human22)
- 능력단위 요소: LM2001020211__16v4

---

### 학습목표(아래)
- 애플리케이션 설계를 기반으로 개발에 필요한 환경을 구성하고, 프로그래밍 언어와 도구를 활용하여 공통 모듈, 업무 프로그램과 배치 프로그램을 구현할 수 있다.

### 핵심키워드(아래)
- 요구 사항 분석, 시스템 아키텍처, 애플리케이션 아키텍처, 공통 모듈, 서버, 형상관리, 단위테스트

### 개발환경 구축하기(아래)
- 개발환경 준비: oracle설치, 계정관리, 용량관리, 권한관리, 테이블관리
- 개발환경 구축: 오라클포트설정, sql developer설치, 테이블시퀸스 생성,Git Hub 저장소 생성

### 공통 모듈 구현하기(아래)
- 공통 모듈 구현: 패키지 분류, class네이밍, Interface로 객체의 캡슐화,  MVC 클래스 패턴사용, View파일 header, footer 분리
- 공통 모듈 테스트(DB클래스 로드,DB접속): Junit 테스트

### 서버 프로그램 구현하기(아래)
- 스프링시큐리티 적용, 비정상적인 접속에 대한 예외 대처, 로그램 예외처리
- 업무 프로세스 확인: 비즈니스 로직 구조(인터페이스에서 DAO, VO데이터 전송)
- 서버 프로그램 구현: Mybatis를 이용하여, 게시판 CRUD DAO를 구현
- 서버 프로그램 테스트: Junit 테스트로 DB클래스 로드, DB접속 및 인터페이스로 CRUD Junit 테스트
### 배치 프로그램 구현하기(아래)
- 배치 프로그램 구현: 회원테이블에서 6개월 동안 회원정보 수정이 없는 회원정보를 콘솔에 출력하는 배치 프로그램 구현
- 배치 프로그램 테스트: DB커넥션, 예외처리, DAO, Interface, console log 확인
- 배치 프로그램 작업순서(아래)

```
### 6개월동안 회원정보 수정이 없는 회원목록 출력

1. 기존 스프링MVC웹프로젝트에 의존성설정 pom.xml에 내용 추가
============================================================
<!-- quartz 라이브러러리 배치작업 -->
	<dependency>
	    <groupId>org.quartz-scheduler</groupId>
	    <artifactId>quartz</artifactId>
	    <version>2.3.0</version>
	</dependency>
<!-- 스프링 Quartz 도우미 라이브러리 배치작업 -->
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-context-support</artifactId>
	    <version>4.3.4.RELEASE</version>
	</dependency>

2. 환경설정 root-context.xml 파일에 배치작업+스케줄 설정 추가
=============================================================
<!-- 배치작업 -->
	<bean id="testJob" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
	    <!-- 서비스 구현 객체의 빈 이름을 인자로 줍니다. -->
	    <property name="targetObject" ref="memberServiceImpl" />
	    <!-- 서비스 객체에서 주기적으로 실행될 메소드른 지정합니다. -->
	    <property name="targetMethod" value="testJobMethod" />
	    <!-- 동시 실행을 방지합니다. -->
	    <property name="concurrent" value="false" />
	</bean>
	<bean id="testJobTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
	    <property name="jobDetail" ref="testJob" />
	    <!-- 시작하고 1분후에 실행한다. (milisecond) -->
	    <property name="startDelay" value="60000" />
	    <!-- 매 24시간마다 실행한다.864000000 (milisecond) -->
	    <property name="repeatInterval" value="60000" />
	</bean>
	<!-- 배치작업 테스트용 스케줄러 -->
	<bean id="testJobScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
	    <property name="triggers">
	        <!-- 앞에서 설정한 트리거를 등록합니다. 필요하면 여러개 만들어서 등록하면 됩니다. -->
	        <list><ref bean="testJobTrigger" /></list>
	    </property>
	</bean>

3. 6개월동안 회원정보 수정이 없는 회원목록 출력 쿼리 추가(마이바티스 mapper 수정 mappers/memberMapper.xml )
=========================================================
<!-- 배치작업 오라클용 getTime() 6개월 동안 회원정보 수정이 없는 회원 -->
  <select id="listMembersTime" resultType="org.edu.vo.MemberVO">
	  <![CDATA[ 
      select 
	    * 
	  from 
	    tbl_member 
	  where UPDATE_DATE < ADD_MONTHS(sysdate,-6)
	  ]]>
  </select>

4. 6개월동안 회원정보 수정이 없는 회원목록 출력을 위한 DAO 인터페이스+임플리먼트 추가
=====================================================================================
public List<MemberVO> getMembersTime() throws Exception; //배치작업 6개월 동안 회원정보 수정이 없는 회원정보
//배치작업 6개월 동안 회원정보 수정이 없는 회원정보
    @Override
    public List<MemberVO> getMembersTime() {
	return sqlSession.selectList(namespace + ".listMembersTime");
    }

5. 6개월동안 회원정보 수정이 없는 회원목록 출력을 위한 서비스 인터페이스+임플리먼트 추가
=======================================================================================
/* 배치작업 */
public void testJobMethod() throws Exception;
/* 배치작업 */
    @Override
    public void testJobMethod() throws Exception {
	List<MemberVO> list = memberDAO.getMembersTime();
	for(MemberVO memberVO : list) {
		System.out.println(memberVO.getUser_id() + ":" + memberVO.getEmail());
	}
    	System.out.println("6개월 동안 회원정보 수정이 없는 회원정보....");
    }

6. 실행
========
웹서버를 실행하면 스케줄에 따라서 배치프로그램이 자동 실행됨
```
### 파스타 클라우드 신청하기(신청완료 후 클라우드 Mysql과 스프링프로젝트 연동하기 실습)
- 파스타 서비스 정상제공됩니다. 신규 신청이 필요합니다. https://paas-ta.kr/experience/apply 
- http://playpark.paas-ta.org/
- 첫째로 작업한 스프링 프로젝트(Hsql용)을 이클립스에서 파스타로 배포 합니다.
- 둘째로 위에서 성공한 소스를 Mysql용으로 변경한 후 이클립스에서 파스타로 배포 합니다.(주, 프로젝트명에 _ 언더바가 있으면 않됩니다.)

### 클라우드 Mysql 원격제어하기: 클라우드 파스타 php앱 추가 후 배포 소스[download this](git_img/myadmin.zip)
- 핵심소스: config.inc.php, libraries/classes/Plugins/Auth/AuthenticationCookie.php
- 클라우드에 배포: php프로젝는 Java프로젝트처럼 이클립스 플러그인으로 배포할 수 없기 때문에 아래처럼 배포
- 우선 윈도우용 cf cli 를 설치한다. https://github.com/cloudfoundry/cli#downloads (아래 직접 다운로드 있음)
- 직접다운로드 [download this](git_img/cf-cli-installer_6.51.0_winx64.zip)

```
터미널 상태에서 phpmyadmin 프로젝트로 이동
>cf login
API endpoint: https://api.paas-ta.org
Email> boramcom@daum.net (본인 파스타 이메일)
Password> (본인 파스타 패스워드)
Authenticating...
OK
>cf push 또는 cf push -s cflinuxfs3 (cflinuxfs3으로 스택을 지정해서 배포)
```

### 서버프로그램 미션
- 6개월 동안 회원정보 수정이 없는 회원 휴면계정으로 업데이트 하기

```
//추가한 코딩(MemberServiceImpl.java파일)
System.out.println("===========6개월간 회원정보 수정없는 회원 휴면계정으로 만들기");
for(MemberVO vo:memberDAO.testJobMethod()) {
	vo.setEnabled(false);
	memberDAO.updateMember(vo);
}
```
### 전자정부표준프레임워크 [관리자관리]메뉴의 CRUD 기능 추가
- 기존 구조파악(ERD, 물리Table, 메이븐pom, web.xml이후 순서 따라감...)
- Hsql DB -> Mysql 로 마이그레이션( 주의, '0000-00-00 00:00:00' DDL문에서 default 일시를 now()함수로 변경.)
- 기본 sht(심플홈템플릿)에서 추가하거나 수정한 파일들(아래)
- ----------------- 물리DB 부분(아래)

```
# 사용자정보 테이블 확인(일반회원+업무사용자+기업회원 중 업무사용자테이블 사용)
# 업무사용자테이블명 = lettnemplyrinfo
# 회원가입시 공통코드 테이블과 상세 테이블에 회원상태값 넣기(아래)
INSERT INTO lettccmmncode VALUES (
'COM013', '회원상태', '회원 가입 신청/승인/삭제를 위한 상태 구분', 'Y', 'LET', now(), 'SYSTEM', now(), 'SYSTEM'
);
INSERT INTO lettccmmndetailcode VALUES ('COM013', 'A', '회원 가입 신청 상태', '회원 가입 신청 상태', 'Y', now(), 'SYSTEM', now(), 'SYSTEM');
INSERT INTO lettccmmndetailcode VALUES ('COM013', 'D', '회원 가입 삭제 상태', '회원 가입 삭제 상태', 'Y', now(), 'SYSTEM', now(), 'SYSTEM');
INSERT INTO lettccmmndetailcode VALUES ('COM013', 'P', '회원 가입 승인 상태', '회원 가입 승인 상태', 'Y', now(), 'SYSTEM', now(), 'SYSTEM');

# ID Generation Service(AI)를 활용하기 위해서 Sequence 저장테이블인 COMTECOPSEQ에 USRCNFRM_ID 항목을 추가해야 한다. 테이블이 생성되어 있는 경우라면 인서트 구문만을 수행한다.(본시스템의 기능 중에서 일반회원, 기업회원관리에서도 USRCNFRM_ID항목을 사용하여 자동증가 고유아이디를 생성한다. 관련정보: https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:%EC%82%AC%EC%9A%A9%EC%9E%90_%EA%B4%80%EB%A6%AC )
CREATE TABLE comtecopseq (
    TABLE_NAME VARCHAR(20) NOT NULL,
    NEXT_ID DECIMAL(30 , 0 ) DEFAULT NULL,
    PRIMARY KEY (TABLE_NAME),
    UNIQUE KEY COMTECOPSEQ_PK (TABLE_NAME)
)  ENGINE=INNODB DEFAULT CHARSET=UTF8;
INSERT INTO `comtecopseq` VALUES ('USRCNFRM_ID',1);

```
- ----------------- 설정 부분(아래)
- pom.xml(mysql 사용 및 log4jdbc 드라이버사용)
- src/main/resources/egovframework/egovProps/globals.properties 설정파일 수정.
- 유니코드 인코딩/디코딩 확인: https://www.online-toolz.com/langs/ko/tool-ko-text-unicode-entities-convertor.html
- src/main/resources/egovframework/spring/com/context-datasource.xml 설정파일 수정.
- ----------------- iBatis와 myBatis 차이
- egov 기본: 아이바티스 ( context-sqlMap.xml ) 
- 아이바티스설명(sqlmap 폴더사용): https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte3.5:psl:data_access
- egov 확장: 마이바티스 ( context-mapper.xml 추가필요 )
- 마이바티스설명(mapper 폴더사용): https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte2:psl:dataaccess:mybatisguide
- 아이바티스 vs 마이바티스 비교: https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte2:psl:dataaccess:ibatis_vs_mybatis
- ------------------ 쿼리 부분(아래)
- src/main/resources/egovframework/sqlmap/config/mysql/sql-map-config-mysql-uss.xml 쿼리위치파일 추가.
- src/main/resources/egovframework/sqlmap/let/uss/umt/EgovMberManage_SQL_Mysql.xml 쿼리파일 추가.
- ------------------ 서비스 + 컨트롤러 클래스 부분(아래)
- egovframework.let.uss.umt.~ 시작하는 자바 패키지 3개 추가
- ------------------ 뷰 css,js 부분(아래)
- src/main/webapp/css/default.css 추가
- src/main/webapp/js/EgovZipPopup.js
- ------------------ 뷰 스프링 설정 수정(아래)
- src/main/webapp/WEB-INF/config/egovframework/springmvc/egov-com-servlet.xml
```
<!-- 패키지 내 Controller, Service, Repository 클래스의 auto detect를 위한 mvc 설정 -->
<context:component-scan base-package="egovframework,edu">
	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
<!-- 로그인 체크가 필요한 URL과 로그인 여부를 체크해준다 -->
    <mvc:interceptors>
        <mvc:interceptor>
		...
		<mvc:mapping path="/uss/umt/mber/*.do"/>
		...
		</mvc:interceptor>
    </mvc:interceptors>		
```
- ------------------ 뷰 jsp 부분(아래)
- src/main/webapp/WEB-INF/jsp/cmm/uss/umt/6개jsp파일
- src/main/webapp/WEB-INF/jsp/main/inc/EgovIncLeftmenu.jsp (메뉴추가)

```
<li class="dept02"><a href="javascript:fn_main_headPageAction('57','uss/umt/mber/EgovMberManage.do')">관리자관리</a></li>
```
작업결과 소스: [download this](git_img/home.zip)
데이터베이스쿼리: [download this](git_img/hst20200808.sql)
데이터베이스ERD(워크벤치): [download this](git_img/simple_home.mwb)

### 전자정부 프로젝트에 Junit 테스터 만들기
- pom.xml에 의존성 추가(아래)

```
<!-- Test 참조 https://offbyone.tistory.com/155 -->
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.12</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>3.0.1</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>4.3.22.RELEASE</version>
 </dependency>
 <dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>4.3.22.RELEASE</version>
</dependency>
<dependency>
	<groupId>javax.inject</groupId>
	<artifactId>javax.inject</artifactId>
	<version>1</version>
</dependency>
```
- 스프링설정중 매퍼설정 추가(context-mapper.xml)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">
    <!-- 실행환경에서 빈이름 참조(EgovAbstractDAO) -->
	<bean id="egov.lobHandler" class="org.springframework.jdbc.support.lob.DefaultLobHandler" lazy-init="true" />
	<!-- Mybatis setup for Mybatis Database Layer -->
	<bean id="egov.sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">		
		<property name="dataSource" ref="egov.dataSource"/>
		<property name="configLocation" value="classpath:/egovframework/mapper/config/mapper-config.xml" />
		<property name="mapperLocations">
			<list>
				<value>classpath:/egovframework/mapper/com/**/*_${Globals.DbType}.xml</value>
			</list>
		</property>
	</bean>
	<!-- Mybatis Session Template -->
	<bean id="egov.sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg ref="egov.sqlSession"/>
	</bean>
</beans>
```
- 매퍼mapper-config.xml 파일 추가

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!--Mybatis 설정 -->
	<settings>
		<!-- 전통적인 데이터베이스 컬럼명 형태인 A_COLUMN을 CamelCase형태의 자바 프로퍼티명 형태인 aColumn으로 자동으로 매핑하도록 함 -->
		<setting name="mapUnderscoreToCamelCase" value="true"></setting>
		<!--  파라미터에 Null 값이 있을 경우 에러 처리 -->
		<setting name="jdbcTypeForNull" value="VARCHAR"></setting>
	</settings>
</configuration>
```
- 쿼리 추가 및 EmployerInfoVO.java 추가(적당한 패키지 생성 후 내부에 생성)

- 매퍼클래스 상속해서 sql세션팩토리 기능 확장

```
/**
 *
 */
package edu.human.com.member.service.impl;

import java.util.List;
import java.util.Map;

import javax.annotation.Resource;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;
import org.apache.ibatis.session.SqlSessionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import egovframework.rte.psl.dataaccess.EgovAbstractMapper;
/**
 * EgovComAbstractMapper.java 클래스
 *
 * @author 서준식
 * @since 2011. 9. 23.
 * @version 1.0
 * @see
 *
 * <pre>
 * << 개정이력(Modification Information) >>
 *
 *   수정일      수정자           수정내용
 *  -------    -------------    ----------------------
 *   2011. 9. 23.   서준식        최초 생성
 *   2016. 5. 11.   장동한        myBatis 방식 적용
 *   2017. 12.30.	김일국	 클래스명 변경
 * </pre>
 */
public abstract class EgovComAbstractMapper extends EgovAbstractMapper{

	private final Logger LOGGER = LoggerFactory.getLogger(this.getClass());
	
	@Resource(name="egov.sqlSession")
	public void setSqlSessionFactory(SqlSessionFactory sqlSession) {
		super.setSqlSessionFactory(sqlSession);
	}
	
	/**
	 * 입력 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId -  입력 처리 SQL mapping 쿼리 ID
	 *
	 * @return DBMS가 지원하는 경우 insert 적용 결과 count
	 */
	@Override
	public int insert(String queryId) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().insert(queryId);
	}

	/**
	 * 입력 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId -  입력 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 입력 처리 SQL mapping 입력 데이터를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 *
	 * @return DBMS가 지원하는 경우 insert 적용 결과 count
	 */
	@Override
	public int insert(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		return getSqlSession().insert(queryId, parameterObject);
	}

	/**
	 * 수정 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 수정 처리 SQL mapping 쿼리 ID
	 *
	 * @return DBMS가 지원하는 경우 update 적용 결과 count
	 */
	@Override
	public int update(String queryId) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().update(queryId);
	}

	/**
	 * 수정 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 수정 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 수정 처리 SQL mapping 입력 데이터(key 조건 및 변경 데이터)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 *
	 * @return DBMS가 지원하는 경우 update 적용 결과 count
	 */
	@Override
	public int update(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		return getSqlSession().update(queryId, parameterObject);
	}

	/**
	 * 삭제 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 삭제 처리 SQL mapping 쿼리 ID
	 *
	 * @return DBMS가 지원하는 경우 delete 적용 결과 count
	 */
	@Override
	public int delete(String queryId) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().delete(queryId);
	}

	/**
	 * 삭제 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 삭제 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 삭제 처리 SQL mapping 입력 데이터(일반적으로 key 조건)를  세팅한 파라메터 객체(보통 VO 또는 Map)
	 *
	 * @return DBMS가 지원하는 경우 delete 적용 결과 count
	 */
	@Override
	public int delete(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		return getSqlSession().delete(queryId, parameterObject);
	}

	//CHECKSTYLE:OFF
	/**
	 * 명명규칙에 맞춰 selectOne()로 변경한다.
	 * @deprecated select() 메소드로 대체
	 * 
	 * @see EgovAbstractMapper.selectOne()
	 */
	//CHECKSTYLE:ON
	@Deprecated
	public Object selectByPk(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectOne(queryId, parameterObject);
	}

	/**
	 * 단건조회 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 단건 조회 처리 SQL mapping 쿼리 ID
	 *
	 * @return 결과 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 단일 결과 객체(보통 VO 또는 Map)
	 */
	@Override
	public <T> T selectOne(String queryId) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectOne(queryId);
	}

	/**
	 * 단건조회 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 단건 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 단건 조회 처리 SQL mapping 입력 데이터(key)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 *
	 * @return 결과 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 단일 결과 객체(보통 VO 또는 Map)
	 */
	@Override
	public <T> T selectOne(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		return getSqlSession().selectOne(queryId, parameterObject);
	}

	/**
	 * 결과 목록을 Map 을 변환한다.
	 * 모든 구문이 파라미터를 필요로 하지는 않기 때문에, 파라미터 객체를 요구하지 않는 형태로 오버로드되었다.
	 *
	 * @param queryId - 단건 조회 처리 SQL mapping 쿼리 ID
	 * @param mapKey - 결과 객체의 프로퍼티 중 하나를 키로 사용
	 *
	 * @return 결과 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 단일 결과 객체(보통 VO 또는 Map)의 Map
	 */
	@Override
	public <K, V> Map<K, V> selectMap(String queryId, String mapKey) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectMap(queryId, mapKey);
	}

	/**
	 * 결과 목록을 Map 을 변환한다.
	 * 모든 구문이 파라미터를 필요로 하지는 않기 때문에, 파라미터 객체를 요구하지 않는 형태로 오버로드되었다.
	 *
	 * @param queryId - 단건 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 맵 조회 처리 SQL mapping 입력 데이터(조회 조건)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 * @param mapKey - 결과 객체의 프로퍼티 중 하나를 키로 사용
	 *
	 * @return 결과 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 단일 결과 객체(보통 VO 또는 Map)의 Map
	 */
	@Override
	public <K, V> Map<K, V> selectMap(String queryId, Object parameterObject, String mapKey) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectMap(queryId, parameterObject, mapKey);
	}

	/**
	 * 결과 목록을 Map 을 변환한다.
	 * 모든 구문이 파라미터를 필요로 하지는 않기 때문에, 파라미터 객체를 요구하지 않는 형태로 오버로드되었다.
	 *
	 * @param queryId - 단건 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 맵 조회 처리 SQL mapping 입력 데이터(조회 조건)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 * @param mapKey - 결과 객체의 프로퍼티 중 하나를 키로 사용
	 * @param rowBounds - 특정 개수 만큼의 레코드를 건너띄게 함
	 *
	 * @return 결과 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 단일 결과 객체(보통 VO 또는 Map)의 Map
	 */
	@Override
	public <K, V> Map<K, V> selectMap(String queryId, Object parameterObject, String mapKey, RowBounds rowBounds) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectMap(queryId, parameterObject, mapKey, rowBounds);
	}

	//CHECKSTYLE:OFF
	/**
	 * 명명규칙에 맞춰 selectList()로 변경한다.
	 * 
	 * @see EgovAbstractMapper.selectList()
	 * @deprecated List<?> 메소드로 대체
	 */
	//CHECKSTYLE:ON
	@Deprecated
	public List<?> list(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectList(queryId, parameterObject);
	}

	/**
	 * 리스트 조회 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 리스트 조회 처리 SQL mapping 쿼리 ID
	 *
	 * @return 결과 List 객체 - SQL mapping 파일에서 지정한  resultType/resultMap 에 의한 결과 객체(보통 VO 또는 Map)의 List
	 */
	@Override
	public <E> List<E> selectList(String queryId) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectList(queryId);
	}

	/**
	 * 리스트 조회 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 리스트 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 리스트 조회 처리 SQL mapping 입력 데이터(조회 조건)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 *
	 * @return 결과 List 객체 - SQL mapping 파일에서 지정한  resultType/resultMap 에 의한 결과 객체(보통 VO 또는 Map)의 List
	 */
	@Override
	public <E> List<E> selectList(String queryId, Object parameterObject) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		return getSqlSession().selectList(queryId, parameterObject);
	}

	/**
	 * 리스트 조회 처리 SQL mapping 을 실행한다.
	 *
	 * @param queryId - 리스트 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 리스트 조회 처리 SQL mapping 입력 데이터(조회 조건)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 * @param rowBounds - 특정 개수 만큼의 레코드를 건너띄게 함
	 *
	 * @return 결과 List 객체 - SQL mapping 파일에서 지정한  resultType/resultMap 에 의한 결과 객체(보통 VO 또는 Map)의 List
	 */
	@Override
	public <E> List<E> selectList(String queryId, Object parameterObject, RowBounds rowBounds) {
		LOGGER.debug("queryId = "+queryId);
		return getSqlSession().selectList(queryId, parameterObject, rowBounds);
	}

	/**
	 * 부분 범위 리스트 조회 처리 SQL mapping 을 실행한다.
	 * (부분 범위 - pageIndex 와 pageSize 기반으로 현재 부분 범위 조회를 위한 skipResults, maxResults 를 계산하여 ibatis 호출)
	 *
	 * @param queryId - 리스트 조회 처리 SQL mapping 쿼리 ID
	 * @param parameterObject - 리스트 조회 처리 SQL mapping 입력 데이터(조회 조건)를 세팅한 파라메터 객체(보통 VO 또는 Map)
	 * @param pageIndex - 현재 페이지 번호
	 * @param pageSize - 한 페이지 조회 수(pageSize)
	 *
	 * @return 부분 범위 결과 List 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 부분 범위 결과 객체(보통 VO 또는 Map) List
	 */
	@Override
	public List<?> listWithPaging(String queryId, Object parameterObject, int pageIndex, int pageSize) {
		LOGGER.debug("queryId = "+queryId);
		LOGGER.debug("parameterObject = "+parameterObject);
		int skipResults = pageIndex * pageSize;
		//int maxResults = (pageIndex * pageSize) + pageSize;

		RowBounds rowBounds = new RowBounds(skipResults, pageSize);

		return getSqlSession().selectList(queryId, parameterObject, rowBounds);
	}

	/**
	 * SQL 조회 결과를 ResultHandler를 이용해서 출력한다.
	 * ResultHandler를 상속해 구현한 커스텀 핸들러의 handleResult() 메서드에 따라 실행된다.
	 *
	 * @param queryId - 리스트 조회 처리 SQL mapping 쿼리 ID
	 * @param handler - 조회 결과를 제어하기 위해 구현한 ResultHandler
	 * @return
	 *
	 * @return 결과 List 객체 - SQL mapping 파일에서 지정한 resultType/resultMap 에 의한 결과 객체(보통 VO 또는 Map)의 List
	 */
	@Override
	public void listToOutUsingResultHandler(String queryId, ResultHandler handler) {
		LOGGER.debug("queryId = "+queryId);
		getSqlSession().select(queryId, handler);
	}
}
```
- Junit 코드

```
package kimilguk_egov;

import java.sql.Connection;
import java.util.List;
import javax.inject.Inject;
import javax.sql.DataSource;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import timespace.miniplugin.authorrole.AuthorRoleDAO;
import timespace.miniplugin.authorrole.AuthorRoleVO;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { 
		"file:src/main/webapp/WEB-INF/config/egovframework/springmvc/egov-com-servlet.xml",
		"file:src/main/resources/egovframework/spring/com/*.xml"} 
)
@WebAppConfiguration
public class DaoTest {

	@Inject
	private DataSource ds;
	
	@Inject
	private AuthorRoleDAO dao;
	
	@Test
	public void testJuit() throws Exception {
		System.out.println("Junit 성공");
	}
	
	@Test
	public void testConnection() throws Exception {
		Connection con = ds.getConnection();
		System.out.println("데이터베이스 커넥션 성공 : " + con);
	}
	
	/*
	 @Test
	public void testSelectEmplyr() throws Exception {
		System.out.println("회원리스트 입니다.");
		EmployerInfoVO employerInfoVO = new EmployerInfoVO();
		List<EmployerInfoVO> list = dao.selectEmplyrList(employerInfoVO);
		System.out.println(list);
		for(EmployerInfoVO vo:list) {
			System.out.println("아이디 : " + vo.getEmplyrId());
			System.out.println("이름 : " + vo.getUserNm());
		}
	}
	 */
}
```
### 미션1: 권한그룹코드 CRUD 만들기(마이바티스사용)
- 권한그룹관리 메뉴 추가
- 회원등록에 사용되는 groupId_result 값 출력이 목표
- 권한등급및회원테이블관계(아래)
![ex_screenshot](./git_img/auth_member.jpg)

### 미션2: 공통코드관리 CRUD 만들기(마이바티스사용)
- 공통코드관리 메뉴 추가
- 회원등록에 사용되는 mberSttus_result 값 출력이 목표
- 공통코드에 사용된 테이블관계(아래)
![ex_screenshot](./git_img/com_code.jpg)

### 참고자료 출처(아래)
- 강사자료: http://blog.daum.net/web_design/search/new%EC%A0%84%EC%9E%90%EC%A0%95%EB%B6%80%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8D
- 학습모듈: https://ncs.go.kr/unity/th03/ncsSearchMain.do 20.정보통신 > 01.정보기술 > 02.정보기술개발 > 02.응용SW엔지니어링
- 배치프로그램 적용기술 참조:(아래)
 
```
https://offbyone.tistory.com/256 ( spring_quartz.zip )

https://minwoohi.tistory.com/89
https://minwoohi.tistory.com/88?category=833256

https://blog.naver.com/PostView.nhn?blogId=p952973&logNo=221011408841
https://blog.naver.com/PostView.nhn?blogId=p952973&logNo=221011723832&parentCategoryNo=&categoryNo=&viewDate=&isShowPopularPosts=false&from=postView

https://cheese10yun.github.io/spring-batch-basic/
```