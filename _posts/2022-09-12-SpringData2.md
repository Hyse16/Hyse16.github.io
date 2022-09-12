---
layout: single
title: "공통 인터페이스 기능"
categories: SpringDataJPA
toc: true
author_profile: true
sidebar:
    nav: "docs"
---
# 순수 JPA 기반 리포지토리 만들기

기본 CRUD
- 저장
- 변경 변경감지 사용 삭제
- 전체 조회
- 단건 조회
- 카운트

````java
//회원//
@Repository
public class MemberJpaRepository {

        @PersistenceContext
        private EntityManager em;

        public Member save(Member member) {
          em.persist(member);
          return member;
        }

      public void delete(Member member) {
          em.remove(member);
        }

      public List<Member> findAll() {
          return em.createQuery
          ("select m from Member m", Member.class)
                  .getResultList();
        }

      public Optional<Member> findById(Long id) {
          Member member = em.find(Member.class, id);
          return Optional.ofNullable(member);
        }

      public long count() {
          return em.createQuery
          ("select count(m) from Member m", Long.class)
                  .getSingleResult();
        }

      public Member find(Long id) {
          return em.find(Member.class, id);
        }
}
````

<br>

````java
//팀//
@Repository
public class TeamJpaRepository {

      @PersistenceContext
      private EntityManager em;

      public Team save(Team team) {
          em.persist(team);
          return team;
}

      public void delete(Team team) {
          em.remove(team);
}

      public List<Team> findAll() {
          return em.createQuery
          ("select t from Team t", Team.class)
                  .getResultList();
}

      public Optional<Team> findById(Long id) {
          Team team = em.find(Team.class, id);
          return Optional.ofNullable(team);
}

      public long count() {
          return em.createQuery
          ("select count(t) from Team t", Long.class)
                  .getSingleResult();
}
}

````

<br>

````java
//test//
@SpringBootTest
@Transactional
public class MemberJpaRepositoryTest {
    
    @Autowired
    MemberJpaRepository memberJpaRepository;
    
    @Test
    public void testMember() {
        
        Member member = new Member("memberA");
        Member savedMember = memberJpaRepository.save(member);
        Member findMember = memberJpaRepository.find(savedMember.getId());

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername()); assertThat(findMember).isEqualTo(member); 

    }
    @Test
    public void basicCRUD() {
        Member member1 = new Member("member1");
        Member member2 = new Member("member2");
        memberJpaRepository.save(member1);
        memberJpaRepository.save(member2);

          Member findMember1 =
  memberJpaRepository.findById(member1.getId()).get();

          Member findMember2 =
  memberJpaRepository.findById(member2.getId()).get();

          assertThat(findMember1).isEqualTo(member1);
          assertThat(findMember2).isEqualTo(member2);

          List<Member> all = memberJpaRepository.findAll(); 
          assertThat(all.size()).isEqualTo(2);

          long count = memberJpaRepository.count(); 
          assertThat(count).isEqualTo(2);

          memberJpaRepository.delete(member1);
          memberJpaRepository.delete(member2);
          
          long deletedCount = memberJpaRepository.count();
          assertThat(deletedCount).isEqualTo(0);
      }

````

<br>

# 공통 인터페이스 구성
![2](/images/2022-09-12-SpringData2/2.png)