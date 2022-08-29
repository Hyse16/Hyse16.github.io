---
layout: single
title: "프록시와 연관관계 관리 "
categories: StudyJPAbasic
toc: true 
author_profile: true
sidebar:
    nav: "docs"
---

# 프록시
em.find() vs em.getReference()

- em.find() : 데이터베이스를 통해서 실제 엔티티 객체 조회

- em.getReference() : 데이터베이스 조회를 미루는 가짜 엔티티 객체 조회

<br>

## 프록시 특징
  

  실제 클래스를 상속 받아서 만들어짐
  
  실제 클래스와 겉 모양이 같음
  
  사용하는 입장에서는 진짜 객체이닞 프록시 객체인지 구분하지 않고 
  사용하면 됨
  
  프록시 객체는 실체 객체의 참조를 보관
  
  프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

<br>

## 프록시 객체의 초기화
![9](/images/2022-08-29-fetchLazy/9.png)


<br>


## 특징
- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화되면 프록시 객체를 통해 실제 엔티티에 접근 가능
- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함(instance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생


<br>

## 프록시 확인
### 프록시 인스턴스의 초기화 여부확인
- PersistenceUnitUtill.isLoaded(Object entity)

### 프록시 클래스 확인 방법
- entity.getClass().getName()출력

### 프록시 강제 초기화
- org.hibernate.Hibernate.iniitialize(entity);

### 참고 : JPA 표준은 강제 초기화 없음, 강제 호출 : member.getName()

<br>



# 즉시 로딩과 지연 로딩
![16](/images/2022-08-29-fetchLazy/16.png)

![20](/images/2022-08-29-fetchLazy/20.png)



<br>

## 프록시와 즉시로딩 주의
- 가급적 지연 로딩만 사용
- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생
- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정
- @OneToMany, @ManyToMany는 기본이 지연 로딩

<br>


# 영속성 전이 : CASCADE
특정 엔티티를 영속 상태를 만들 때 연관된 엔티티도 함께 영속상태로 만들고 싶을 때
ex) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

<br>

![29](/images/2022-08-29-fetchLazy/29.png)

<br>

## 주의!
영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음

엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐

<br>


## CASCADE의 종류
- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제
- MERGE : 병합
- REFRESH : REFRESH
- DETACH : DETACH

<br>

# 고아 객체
고아 객제 체거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제

orphanRemoval = true

<br>

## 주의!
참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능

참조하는 곳이 하나일 때 사용해야 함

특정 엔티티가 개인 소유할 때 사용

@OneToOne, @OneToMany만 가능

<br>

# 영속성 전이 + 고아 객체, 생명주기
Cascade Type.ALL+orphanRemovel=true

스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거

두 옵션을 모두 활성화 화면 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있음