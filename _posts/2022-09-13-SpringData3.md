---
layout: single
title: "쿼리 메소드 기능"
categories: SpringDataJPA
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

쿼리 메소드 기능 3가지
- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 JPA NamedQuery 호출
- @Query 어노테이션을 사용해서 리파지토리 인터페이스에 쿼리 직접 정의

<br>

# 메소드 이름으로 쿼리 생성

이름과 나이를 기준으로 회원을 조회하려면?

````java
//순수 JPA 리포지토리//
public List<Member>
findByUsernameAndAgeGreaterThan
(String username, int age){
    return em.createQuery("select m from Member m where 
    m.username = :username and m.age > :age")
        .setParameter("username",username)
        .setParameter("age",age)
        .getResultList();
}

//순수 JPA테스트 코드//

@Test
public void findByUsernameAndAgeGreaterThan(){
    Member m1 = new Member("AAA",10);
    Member m2 = new Member("AAA",20);
    memberJPARepository.save(m1);
    memberJPARepository.save(m2);

    List<Member> result = 
    memberJPARepository.findByUsernameAndAgeGreaterThan("AAA",15); 

    assertThat(result.get(0).getUsername()).isEqualTo("AAA");
    assertThat(rersult.get(0).getAge()).isEqualTo(20);
    assertThat(result.size()).isEqualto(1);
}

//스프링데이터JPA//
public interface MemberRepository
 extends JpaRepository<Member, Long> {
    List<Member> 
    findByUsernameAndAgeGreaterThan(Stirng username, int age);
}

/*참고
엔티티 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 변경해야 한다!.*/

````
<br>

# JPA NamedQuery
````java
// @NamedQuery 어노테이션으로 Named 쿼리 정의//
@Entity
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m 
    where m.username =:username")
    public class Member{
            ...
    }


//JPA를 직접 사용해서 Named 쿼리 호출//
public class MemberRepository{
    public List<Member> findByUsername(String username){
        ...
        List<Member> resultList = 
        em.createNamedQuery
        ("Member.findByUsername",Member.class)
        .setParameter("username",username)
        .getresultList();
    }
}

//스프링 데이터 JPA로 NamedQuery 사용//

@Query(name = "Member.findByUsername")
List<Member> 
findByUsername(@Param("username") String username);

//스프링 데이터 JPA로 Named 쿼리 호출//

public interface MemberRepository 
extends JpaRepository<Member, Long>{
    List<Member> findByUsername
    (@Param("username") String useranme);
}
````

<br>

# @Query 리포지토리 메소드에 쿼리 정의하기
````java
public interface MemberRepository 
extends JpaRepository<Member, Long>{
    
    @Query("select m from Member m where
     m.useranme =:username and m.age = :age")
    List<Member> findUser
    (@Param("username") String username, 
    @Param("age") int age);
}

````

<br>

## @Query, 값, DTO 조회하기
````java
//하나의 값 조회//
@Query("select m.username from Member m")
List<String> findUsernameList();

//DTO로 직접 조회//
@Query("select new study.datajpa.dto.MemberDto
(m.id, m.username, t.name)"+ 
"from Member m join m.team t")
List<MemberDto> findMemberDto();

//DTO생성//

  @Data
  public class MemberDto {

      private Long id;
      private String username;
      private String teamName;

      public MemberDto(Long id, String username, String teamName) {
          this.id = id;
          this.username = username;
          this.teamName = teamName;
      }
}
````

# 파라미터 바인딩
- 위치 기반
- 이름 기반
  
````java
select m from Member m where m.username = ?0 //위치 기반
select m from Member m where m.username = :name //이름 기반

//파라미터 바인딩//
public interface MemberRepository extends 
JpaRepository<Member, Long>{
    @Query("select m from Member m where 
    m.username = :name")
    Member findMembers(@Param("name") String username);
}

//컬렉션 파라미터 바인딩//
@Query("select m from Member m where 
m.username in :names")
    List<Member> 
    findByNames(@Param("names") List<String> names);
 ````

<br>

# 반환 타입
````java
//스프링 데이터 JPA는 유연한 반환 타입 지원//

List<Member> findByUsername(String name); //컬렉션 Member findByUsername(String name); //단건
Optional<Member> findByUsername(String name); //단건 Optional

````

조회 결과가 많거나 없으면? 

컬렉션
- 결과 없음: 빈 컬렉션 반환

단건 조회
- 결과 없음: null 반환
- 결과가 2건 이상: javax.persistence.NonUniqueResultException 예외 발생