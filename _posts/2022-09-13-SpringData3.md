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

<br>

# 순수 JPA페이징과 정렬

- 검색조건 : 나이가 10살
- 정렬조건 : 이름으로 내림차순
- 페이징조건 : 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

````java
//JPA 페이징리포지토리 코드//
public List<Member> findByPage
(int age, int offset, int limit){
    return em.createQuery
    ("select m from Member m where m.age = :age 
    order by m.username desc")
        .setParameter("age",age)
        .setFirstResult(offset)
        .setMaxResult(limit)
        .getResultList();
}

public long totalCount(int age){
    return em.createQuery
    ("select count(m) from Member m 
    where m.age = :age", Long.class)
        .setParameter("age",age)
        .getSingleResult();
}

//JPA 페이징 테스트 코드//

@Test
public void paging() throws Exception{
    memberJpaRepository.save
    (new Member("member1",10));
    memberJpaRepository.save
    (new Member("member2",10));
    memberJpaRepository.save
    (new Member("member3",10));
    memberJpaRepository.save
    (new Member("member4",10));
    memberJpaRepository.save
    (new Member("member5",10));

    int age = 10;
    int offset = 0;
    int limit = 3;

    List<Member> members = 
    memberJpaRepository.findByPage
    (age, offset, limit);
    long totalCount = 
    memberJpaRepository.totalCount(age);

    assertThat(members.size()).isEqualTo(3);
    assertThat(totalCount).isEqualTo(5);
}
````

<br>

# 스프링 데이터JPA페이징과 정렬
````java
//Page 사용 예제 정의 코드//
public interface MemberRepository extends
 Repository<Member, Long>{
    Page<Member> findByAge(int age, Pageable pageable);
}

//Page 사용 예제 실행 코드//

@Test
public void page() throws Exception{
    memberRepository.save
    (new Member("member1",10));
    memberRepository.save
    (new Member("member2",10));
    memberRepository.save
    (new Member("member3",10));
    memberRepository.save
    (new Member("member4",10));
    memberRepository.save
    (new Member("member5",10));

    PageRequeset pageRequest = 
    PageRequest.of
    (0,3, Sort.by(Sort.Direction.DESC,"username"));

    Page<Member> page =
     memberRepository.findByAge(10, pageRequest);

    List<Member> content = page.getContent();
    assertThat(content.size()).isEqualTo(3);
    assertThat(page.getTotalElements()).isEqualTo(5); 
    assertThat(page.getNumber()).isEqualTo(0); 
    assertThat(page.getTotalPages()).isEqualTo(2);  
    assertThat(page.isFirst()).isTrue(); 
    assertThat(page.hasNext()).isTrue();
}

//주의할 점//
//Page는 0부터 시작이다//

//페이지를 유지하면서 엔티티를 DTO로 변환하기//
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
````

<br>

# 벌크성 수정 쿼리

````java
//JPA를 사용한 벌크성 수정 쿼리//
public int bulkAgePlus(int age){
    int resultCount = em.createQuery{
        "update Member m set m.age = m.age +1"+
        "where m.age >= :age")
        .setParameter("age",age)
        .executeUpdate();
        return resultCount;
    }
}

//JPA를 사용한 벌크성 수정 쿼리 테스트//
@Test
  public void bulkUpdate() throws Exception {

      memberJpaRepository.save
      (new Member("member1", 10));
      memberJpaRepository.save
      (new Member("member2", 19));
      memberJpaRepository.save
      (new Member("member3", 20));
      memberJpaRepository.save
      (new Member("member4", 21));
      memberJpaRepository.save
      (new Member("member5", 40));

      int resultCount = 
      memberJpaRepository.bulkAgePlus(20);

      assertThat(resultCount).isEqualTo(3);
  }
````

<br>

````java
//스프링 데이터 JPA를 사용한 벌크성 수정 쿼리//
@Modifying
@Query("update Member m set m.age = m.age+1 
where m.age >=: age")
int bulkAgePlus(@Param("age") int age);

//스프링 데이터 JPA를 사용한 벌크성 수정 쿼리 테스트//
@Test
  public void bulkUpdate() throws Exception {
//given
      memberRepository.save
      (new Member("member1", 10));
      memberRepository.save
      (new Member("member2", 19));
      memberRepository.save
      (new Member("member3", 20));
      memberRepository.save
      (new Member("member4", 21));
      memberRepository.save
      (new Member("member5", 40));
      
      int resultCount = 
      memberRepository.bulkAgePlus(20);

      assertThat(resultCount).isEqualTo(3);
    }
//벌크성 수정,삭제 쿼리는 @Modifying 어노테이션 사용//
// 사용하지 않으면 예외발생//
/*벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화:
 @Modifying(clearAutomatically = true)*/
````

<br>

# EntityGraph
연관된 엔티티들을 SQL 한번에 조회하는 방법

````java
@Test
public void findMemberLazy() throws Exception{
      Team teamA = new Team("teamA");
      Team teamB = new Team("teamB");
      teamRepository.save(teamA);
      teamRepository.save(teamB);
      memberRepository.save
      (new Member("member1", 10, teamA));
      memberRepository.save
      (new Member("member2", 20, teamB));

      em.flush();
      em.clear();

      List<Member> members = 
      memberRepository.findAll();

      for (Member member : members) {
          member.getTeam().getName();
} 
}
````

<br>

````java
//JPQL 페치 조인//
@Query("select m from Member m 
left join fetch m.team")
List<Member> findMemberFetchJoin();

//EntityGraph 사용//
//공통 메서드 오버라이드//
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.//
@EntityGraph(attributePaths={"team"})
List<Member> findByUsername(String username)
````
정리
- 페치조인의 간편 버전
- LEFT OUTER JOIN 사용

<br>

# JPA HINT

````java
//쿼리 힌트 사용//
@QueryHints(value = @QueryHint
(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);

//쿼리 힌트 사용 확인//
@Test
  public void queryHint() throws Exception {

      memberRepository.save(new Member("member1", 10));
      em.flush();
      em.clear();

      Member member = 
      memberRepository.findReadOnlyByUsername("member1");
      member.setUsername("member2");

    em.flush(); 
}

//쿼리 힌트 Page 추가 예제//
@QueryHints(value = { 
@QueryHint(name = "org.hibernate.readOnly",
               value = "true")},
   forCounting = true)

    Page<Member> findByUsername
    (String name, Pagable pageable);

````