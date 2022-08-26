---
layout: single
title: "엔티티 매핑"
categories: StudyJPAbasic
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

<br>

# 엔티티 매핑 소개
- 객체와 태이블 매핑 : @Entity, @Table
- 필드와 컬럼 매핑 : @Column
- 기본키 매핑 : @Id
- 연관관계 매핑 : @ManyToOne, @JoinColumn
  
<br>

## 객체와 테이블 매핑

### @Entity

- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라고 한다.

- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수

<br>

### 주의할점!!!
- 기본 생성자 필수
- final 클래스, enum, interface 사용 x
- 저장할 필드에 final 사용x

<br>

### @Entity 속성 정리
- 속성 : name
- JPA에서 사용할 엔티티 이름을 지정한다
- 같은 클래스 이름이 없으면 기본값 사용

<br>

### @Table
- @Table은 엔티티와 매핑할 테이블 지정
  
![8](/images/2022-08-26-EntityMapping/8.png)

<br>

## 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성

- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해 데이터베이스에 맞는 적절한 DDL생성
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

<br>


### 데이터베이스 스키마 자동 생성 - 속성

![11](/images/2022-08-26-EntityMapping/11.png)


<br>

### 데이터베이스 스키마 자동 생성  - 주의
- 운영 장비에는 절대 create, create-drop, update 사용 X
- 개발 초기 단계는 create OR update
- 테스트 서버는 update OR validate
- 스테이징과 운영 서버는 validate OR none

<br>

### DDL생성 기능
- 제약조건 추가 : 회원 이름은 필수
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.
  
<br>

## 필드와 컬럼 매핑

### 매핑 어노테이션 정리
- @Column : 컬럼 매핑
- @Temporal : 날짜 타입 매핑
- @Enumerated : enum 타입 매핑(EnumType.ORDINAL 사용!!!)
- @Lob : BLOB, CLOB 매핑
- @Transient : 특정 필드를 컬럼에 매핑하지 않음


<br>

## 기본 키 매핑
- @Id
- @GeneratedValue

<br>

### 기본 키 매핑 방법
- 직접 할당 : @Id만 사용
- 자동 생성 : @GeneratedValue 사용

