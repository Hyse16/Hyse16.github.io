---
layout: single
title: "SpringData 동작확인"
categories: SpringDataJPA
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

# 실제 동작하는지 테스트하기
````java

//회원 엔티티//
@Entity
@Getter @Setter
    public class Member {

        @Id @GeneratedValue
        private Long id;
        private String username;
}

//회원 JPA 리포지토리//
@Repository
public class MemberJpaRepository {

        @PersistenceContext
        private EntityManager em;

        public Member save(Member member) {
            em.persist(member);
            return member;
}
        public Member find(Long id) {
            return em.find(Member.class, id);
} 
}

//JPA 기반 테스트//
@SpringBootTest
@Transactional
@Rollback(false)
public class MemberJpaRepositoryTest {

      @Autowired
      MemberJpaRepository memberJpaRepository;

      @Test
      public void testMember() {
          Member member = new Member("memberA");

          Member savedMember = 
          memberJpaRepository.save(member);
          
          Member findMember = memberJpaRepository.find(savedMember.getId());
          
          assertThat(findMember.getId())
          .isEqualTo(member.getId());

          assertThat(findMember.getUsername())
          .isEqualTo(member.getUsername()); 

          assertThat(findMember).isEqualTo(member); 
          //JPA 엔티티 동일성 보장
}
}

````

<br>

# 스프링 데이터 JPA 리포지토리

````java

public interface MemberRepository extends JpaRepository<Member, Long> {

}
   
//스프링 데이터 JPA 기반 테스트//
@SpringBootTest
  @Transactional
  @Rollback(false)
  public class MemberRepositoryTest {
        
        @Autowired
        MemberRepository memberRepository;
        
        @Test
        public void testMember() {
            Member member = new Member("memberA");

            Member savedMember = memberRepository.save(member);

            Member findMember =
    memberRepository.findById
    (savedMember.getId()).get();

            Assertions.assertThat(findMember.getId())
            .isEqualTo(member.getId());
            Assertions.assertThat(findMember.getUsername())
            .isEqualTo(member.getUsername())
            Assertions.assertThat(findMember).isEqualTo(member); 
            //JPA 엔티티 동일성보장
}

````

<br>

# 예제 도메인 모델과 동작확인

````java

//Member 엔티티//
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "username", "age"})
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
    public Member(String username) {
        this(username, 0);
}

    public Member(String username, int age) {
        this(username, age, null);
}

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
}

       public void changeTeam(Team team) {
          this.team = team;
          team.getMembers().add(this);
} 
}

//Team 엔티티//
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "name"})
public class Team {

      @Id @GeneratedValue
      @Column(name = "team_id")
      private Long id;

      private String name;

      @OneToMany(mappedBy = "team")
      List<Member> members = new ArrayList<>();
      public Team(String name) {
          this.name = name;
} 
}

//데이터 확인 테스트//
@SpringBootTest
public class MemberTest {
   
    @PersistenceContext
    EntityManager em;
     
      @Test
      @Transactional
      @Rollback(false)
      public void testEntity() {
          Team teamA = new Team("teamA");
          Team teamB = new Team("teamB");
          em.persist(teamA);
          em.persist(teamB);
          
          Member member1 = new Member("member1", 10, teamA);
          Member member2 = new Member("member2", 20, teamA);
          Member member3 = new Member("member3", 30, teamB);
          Member member4 = new Member("member4", 40, teamB);
          
          em.persist(member1);
          em.persist(member2);
          em.persist(member3);
          em.persist(member4);
            
          //초기화 
          em.flush(); 
          em.clear();

        //확인
          List<Member> members = em.createQuery("select m from Member m",
  Member.class)
                  .getResultList();

          for (Member member : members) {
              System.out.println("member=" + member);
              System.out.println("-> member.team=" + member.getTeam());
        } 
    }
}
````
