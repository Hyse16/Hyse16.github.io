---
layout: single
title: "확장기능"
categories: SpringDataJPA
toc: true
author_profile: true
sidebar:
    nav: "docs"
---
# 사용자 정의 리포지토리 구현
- 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성 
- 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야 하는 기능이 너무 많음 
- 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면?


````java
//사용자 정의 인터페이스//
public interface MemberRepositoryCustom{
    List<Member> findMemberCustom();
}

//사용자 정의 인터페이스 구현 클래스//
@RequiredArgsConstructor
public class MemberRepositoryImpl 
implements MemberRepositoryCustom {

      private final EntityManager em;

      @Override
      public List<Member> findMemberCustom() {
          return em.createQuery
          ("select m from Member m")
                  .getResultList();
} 
}

//사용자 정의 인터페이스 상속//
public interface MemberRepository
           extends JpaRepository<Member, Long>,
            MemberRepositoryCustom {
}

//사용자 정의 메서드 호출 코드//
  List<Member> result = 
  memberRepository.findMemberCustom();
````

사용자 정의 구현 클래스

규칙: 리포지토리 인터페이스 이름 + Impl

스프링 데이터 JPA가 인식해서 스프링 빈으로 등록

<br>

# Audting
엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면? 
- 등록일
- 수정일
- 등록자 
- 수정자

````java
//순수 JPA 사용//
@MappedSuperclass
@Getter
public class JpaBaseEntity {
      @Column(updatable = false)
      private LocalDateTime createdDate;
      private LocalDateTime updatedDate;

      @PrePersist
      public void prePersist() {
          LocalDateTime now = 
          LocalDateTime.now();
          createdDate = now;
          updatedDate = now;
}

      @PreUpdate
      public void preUpdate() {
          updatedDate = LocalDateTime.now();
      }
}
  public class Member extends JpaBaseEntity {} 

// 확인 코드//
@Test
public void JpaEventBaseEntity() throws Exception {

      Member member = new Member("member1");
      memberRepository.save(member); //@PrePersist
      Thread.sleep(100);
      member.setUsername("member2");
      em.flush(); //@PreUpdate
      em.clear();

      Member findMember = memberRepository.
      findById(member.getId()).get();

      System.out.println
      ("findMember.createdDate = " +findMember.getCreatedDate());

      System.out.println
      ("findMember.updatedDate = " +findMember.getUpdatedDate());
  }
````

<br>

````java
//스프링 데이터 Auditing 적용 - 등록일, 수정일//
 @EntityListeners(AuditingEntityListener.class)
 @MappedSuperclass
 @Getter
 public class BaseEntity {
      @CreatedDate
      @Column(updatable = false)
      private LocalDateTime createdDate;

      @LastModifiedDate
      private LocalDateTime lastModifiedDate;
}

//스프링 데이터 Auditing 적용 - 등록자, 수정자//
 @EntityListener(AuditingEntityListener.class)
  @MappedSuperclass
  public class BaseEntity {

      @CreatedDate
      @Column(updatable = false)
      private LocalDateTime createdDate;

      @LastModifiedDate
       private LocalDateTime lastModifiedDate;

      @CreatedBy
      @Column(updatable = false)
      private String createdBy;

      @LastModifiedBy
      private String lastModifiedBy;
  }

  //등록자, 수정자를 처리해주는 AuditorAware 스프링 빈 등록//
   @Bean
   public AuditorAware<String> auditorProvider() {
        return () -> Optional.of
        (UUID.randomUUID().toString());
    }


/* 참고: 실무에서 대부분의 엔티티는 등록시간, 수정시간이 필요하지만,
 등록자, 수정자는 없을 수도 있다.
 그래서 다음과 같이 Base 타입을 분리하고, 
 원하는 타입을 선택해서 상속한다.*/

public class BaseTimeEntity {
        @CreatedDate
        @Column(updatable = false)
        private LocalDateTime createdDate;
        @LastModifiedDate
        private LocalDateTime lastModifiedDate;
}

  public class BaseEntity extends BaseTimeEntity {
        @CreatedBy
        @Column(updatable = false)
        private String createdBy;
        @LastModifiedBy
        private String lastModifiedBy;
  }
````