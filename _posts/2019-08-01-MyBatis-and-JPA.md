---
title: MyBatis 와 JPA 혼합하기 
categories:
- java
excerpt: |
  회사에서 기존에 운영하던 서비스를 개선하는 프로젝트가 생겼다. 옛날 사양이기도 한지라 새로운 기술도 넣어가면서 정돈하는 것이 목표였다. 기존에 사용하던 persistence 레이어는 MyBatis에서 JPA로 변경하기로 한 것! 이전에 참여했던 프로젝트에서 JPA를 적용해봤던지라 그걸 적당히 응용하면 되겠지..했는데 큰 오산이었다. 이미 디비가 RDB여서 JPA 의 철학과 목적에 맞게 구현이 어려워진 것이다.
feature_text: |
  ## MyBatis 와 JPA 혼합하기 
  MyBatis 와 JPA 혼합하기 
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

MyBatis와 JPA 혼합하기
====

회사에서 기존에 운영하던 서비스를 개선하는 프로젝트가 생겼다. 옛날 사양이기도 한지라 새로운 기술도 넣어가면서 정돈하는 것이 목표였다.   
기존에 사용하던 persistence 레이어는 MyBatis에서 JPA로 변경하기로 한 것! 이전에 참여했던 프로젝트에서 JPA를 적용해봤던지라 그걸 적당히 응용하면 되겠지..했는데 큰 오산이었다.  
이미 디비가 RDB여서 JPA 의 철학과 목적에 맞게 구현이 어려워진 것이다. 이전 JPA 적용 프로젝트는 디비 선택과 설계부터 시작한, 맨 땅 다지기 부터 시작한 집짓기였기 때문에 가능했던 것이었다.  
단기간 내에 안정성 있게 구현해야하는 것이 최우선이었기 때문에 결국 JPA에 MyBatis를 혼합한 형태가 되어버렸다.   
와 그럼 datasource는 어떡하지? JPA의 entitymanager는 어떡하지? 어떻게 관리되는거지? 여차저차 세팅까지 해봤는데 결국 작지만 큰 문제가 생겼다. transaction에서 문제가 생긴 것이다.  
JPA의 구동방식과 myBatis의 구동방식이 달라서 내부 관리가 어려웠던 것은 메소드를 구현해서 어찌됐든 해결을 했는데 (그건 기회가 되면 다음번에 정리) 한 메소드 내에서 두 개를 같이 쓸 때 에러가 났다.   
transaction 관련 에러였는데, 결론은 두 개를 같이 쓰면 transactionManager가 관리하지 못하겠다고 한 거였다.  
   
환경 : datasource bean 1, sqlSessionFactory, entityManagerFactory, transactionManager(는 어떻게?)

결론부터 말하자면 JpaTransactionManager 를 사용하면 된다. 처음에는 sqlSessionFactory 설정을 해줄 때 별도의 transactionManager 를 세팅해주고 Jpa용 transactionManager를 세팅해줘서   
문제였는데, JpaTransactionManager를 사용하면  MyBatis를 함께 관리해준다. 다른 transactionManager 와는 다르게 myBatis처럼 직접 datasource로 접근해서 처리할 수 있는 것도 포인트다.  
  
<pre><code>

	@Bean
	public JpaTransactionManager transactionManager(EntityManagerFactory emf) {
		JpaTransactionManager transactionManager = new JpaTransactionManager();
		transactionManager.setEntityManagerFactory(emf);
		
		return transactionManager;
	}
</code></pre>

위와 같이 설정해준다.   

그래도 당쵀 sqlSessionFactory에 어떠한 transactionManager를 설정해주는 곳이 없고, 오히려 null인 경우 Spring이 관리하는 기본 트랜잭션이 세팅되는데 뭘 어떻게 한군데서 관리한단 걸까??  
키워드로 알아본 것은, configuration 클래스 위에 선언한 @EnableTransactionManagement 이 어노테이션이다. 어노테이션을 타고타고 올라가면 transactionManager가 없으면 기본으로 PlatformTransactionManager를 사용한다. 하지만 있으면?  
TransactionManagementConfigurer 클래스에서 어노테이션이 설정된 곳에서 만약 해당 빈을 설정해주면 그것으로 사용하고 아니면 디폴트로 PlatformTransactionManager 를 만들어 사용하고!  
참조 : https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/TransactionManagementConfigurer.html  
  
이렇게 해두면, AbstractTransactionManagementConfiguration 에서 설정된 transactionManager를 가져오고, 주석 내 ProxyTransactionManagementConfiguration 에서 등록된 transactionManager를 사용하게 해준다.     
JpaTransactionManager가 직접 접근이 가능하다고 했으므로 MyBatis, JPA가 함께 관리될 수 있는 것이다.








 



