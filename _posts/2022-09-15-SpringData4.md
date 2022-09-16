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

<br>

# Web 확장 - 도메인 클래스 컨버터

````java
//도메인 클래 컨버터 사용 전//
@RestController
@RequiredArgsConstructor
public class MemberController {

      private final MemberRepository memberRepository;
      @GetMapping("/members/{id}")
      public String findMember
      (@PathVariable("id") Long id) {
          Member member =
           memberRepository.findById(id).get();
          return member.getUsername();
      }
}

//도메인 클래스 컨버터 사용 후//
@RestController
@RequiredArgsConstructor
public class MemberController {

      private final MemberRepository memberRepository;
      @GetMapping("/members/{id}")
      public String findMember
      (@PathVariable("id") Member member) {
          return member.getUsername();
      }
}

````

<br>

# Web 확장 - 페이징과 정렬

````java
//페이징과 정렬 예제//
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
        Page<Member> page = 
        memberRepository.findAll(pageable);
        return page;
    }
 
//기본값, 글로벌 설정 : 스프링 부트//

//# 기본 페이지 사이즈//
spring.data.web.pageable.default-page-size=20 

///# 최대 페이지 사이즈//
spring.data.web.pageable.max-page-size=2000 

//개별설정//
@RequestMapping(value = "/members_page", method = RequestMethod.GET)
  public String list(@PageableDefault
  (size = 12, sort = “username”,
direction = Sort.Direction.DESC) Pageable pageable) {
  ...
}

//접두사//
//페이징 정보가 둘 이상이면 접두사로 구분//
public String list(
      @Qualifier("member") Pageable memberPageable,
      @Qualifier("order") Pageable orderPageable, ...

````

<br>

## Page 내용을 DTO로 변환하기
엔티티를 API로 노출하면 다양한 문제가 발생한다. 

그래서 엔티티를 꼭 DTO로 변환해서 반환해야 한다.

Page는 map() 을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.

````java
//MemberDTO//
@Data
  public class MemberDto {
      private Long id;
      private String username;
      public MemberDto(Member m) {
          this.id = m.getId();
          this.username = m.getUsername();
      }
  }

//Page.Map()사용//
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
      Page<Member> page = 
      memberRepository.findAll(pageable);
      Page<MemberDto> pageDto = 
      page.map(MemberDto::new);
      return pageDto;
}

//코드 최적화//
@GetMapping("/members")
public Page<MemberDto> list
(Pageable pageable) {
      return memberRepository.findAll
      (pageable).map(MemberDto::new);
  }
````