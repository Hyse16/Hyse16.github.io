---
layout: single
title: "스프링데이터 JPA분석"
categories: SpringDataJPA
toc: true
author_profile: true
sidebar:
    nav: "docs"
---
# 스프링 데이터 JPA 구현체 분석
스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체

````java
//리스트 12.31 SimpleJpaRepository//
@Repository
@Transactional(readOnly =true)
public class SimpleJpaRepository<T, ID>...{

    @Transactional
    public <S extends T> S save(S entity){
        if (entityInformaiton.isNew(entity)){
            em.persist(entity);
            return entity;
        }
        else{
            return em.merge(entity);
        }
    }
    ...
}
````
@Transactional 트랜잭션 적용
- JPA의 모든 변경은 트랜잭션 안에서 동작
- 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리
-서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
- 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용
- 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록,
변경이 가능했음(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것임)



@Transactional(readOnly = true)

데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 readOnly = true 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음//

<br>

중요!

save() 메서드
- 새로운 엔티티면 저장(persist)
- 새로운 엔티티가 아니면 병합(merge)

<br>

# 새로운 엔티티를 구별하는 방법
새로운 엔티티를 판단하는 기본 전략
- 식별자가 객체일 때 null 로 판단
- 식별자가 자바 기본 타입일 때 0 으로 판단
- Persistable 인터페이스를 구현해서 판단 로직 변경 가능

````java
//Persistable//
public interface Persistable<ID> {
        ID getId();
        boolean isNew();
    }

//Persistable 구현//
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
  public class Item implements Persistable<String> {

      @Id
      private String id;
      @CreatedDate
      private LocalDateTime createdDate;
      public Item(String id) {
          this.id = id;
}

      @Override
      public String getId() {return id; }
      @Override
      public boolean isNew() {
          return createdDate == null;
      }
}
````