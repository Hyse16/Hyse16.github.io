---
layout: single
title: "영속성 관리"
categories: StudyJPAbasic
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

<br>

# 영속성 컨테스트
- JPA를 이해하는데 가장 중요한 용어
- 엔티티를 영구 저장하는 환경 이라는  뜻
- EntityManage.persist(entity);

<br>

## 엔티티 매니저? 영속성 컨텍스트?
- 영속성 컨텍스트는 논리적 개념
- 눈에 보이지 않는다
- 엔티티 매니저를 통해 영속성 컨텍스트에 접근

<br>

## 엔티티의 생명주기


### 비영속
- 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태

![11](../../images/2022-08-25-Durability/11.png)

<br>

### 영속
- 영속성 컨텍스트에 관리되는 상태
  
![12](../../images/2022-08-25-Durability/12.png)

<br>

### 준영속
- 영속성 컨텍스트에 저장되었다가 분리된 상태
  
![13](../../images/2022-08-25-Durability/13.png)

<br>

### 삭제
- 삭제된 상태
  
![14](../../images/2022-08-25-Durability/14.png)

<br>


<br>

## 영속성 컨텍스트의 이점
- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연
- 변경 감지(Dirty Checking)
- 지연 로딩(Lazy Loading)
  
<br>

## 엔티티 조회, 1차 캐시
  
![15](../../images/2022-08-25-Durability/15.png)

<br>


## 1차 캐시에서 조회
  
![16](../../images/2022-08-25-Durability/16.png)

<br>


## 데이터베이스에서 조회
  
![17](../../images/2022-08-25-Durability/17.png)


<br>

## 영속 엔티티의 동일성 보장
  
![18](../../images/2022-08-25-Durability/18.png)


<br>



## 엔티티 등록(트랜잭션을 지원하는 쓰기 지연)
  
![19](../../images/2022-08-25-Durability/19.png)

<br>

  
![20](../../images/2022-08-25-Durability/20.png)

<br>

  
![21](../../images/2022-08-25-Durability/21.png)


<br>

## 엔티티 수정(변경 감지)
  
![22](../../images/2022-08-25-Durability/22.png)

<br>

  
![23](../../images/2022-08-25-Durability/23.png)

<br>

## 엔티티 삭제

![24](../../images/2022-08-25-Durability/24.png)

<br>

# 플러시
- 영속성 컨텍스트의 변경내용을 데이터베이스에 반영

<br>

## 플러시 발생
- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송

<br>

## 영속성 컨텍스트를 플러시 하는 방법
- em.flush() - 직접 호출
- 트랜잭션 커밋 - 플러시 자동 호출
- JPQL 쿼리 실행 - 플러시 자동 호출

<br>

## 플러시는!!!
- 영속성 컨텍스트를 비우지 않음
- 영속성 컨텍스트이 변경내용을 데이터베이스에 동기화
-  트랜잭션이라는 작업 단위가 중요( 커밋 직전에만 동기화 하면 됨)

<br>

# 준영속 상태
- 영속 -> 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리
- 영속성 컨텍스트가 제공하는 기능을 사용 못함

<br>

## 준영속 상태로 만드는 방법
- em.detach(entity)
  
  특정 엔티티만 준영속 상태로 전환

 - em.clear
   
   영속성 컨텍스트를 완전히 초기화
  
- em.close
  
  영속성 컨텍스트를 종료


