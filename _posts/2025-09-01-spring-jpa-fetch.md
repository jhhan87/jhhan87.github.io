---
title: spring jpa fetch error
description: jpa query fetcj cache error
author: jhhan
date: 2025-09-01 12:50:00 +0900
categories: [Blogging, ErrorNote]
tags: [java, spring boot, jpa]
math: true
mermaid: true
---


## 발생일시
2025년 9월 1일


## 발단
jpa 영속관계로 묶은 부모-자식데이터 조회 시, 자식테이블 데이터를 조회하는데 부모테이블 데이터가 전혀 다른 데이터로 묶여 조회되는 현상 (자동생성 쿼리를 확인하자 id가 이상하게 맵핑된 것을 확인함). 

jpa 자동생성 쿼리를 사용했을 때 발생한 문제였고, @Query annotation을 추가하여 직접 쿼리문을 날리게 하자 정상적으로 부모데이터를 묶어 갖고오는 것 확인함


## 원인
기존 실무에서는 컨벤션으로 '논리적인 관계'라고 하여 부모자식 관계의 테이블이라 해도 실제 연관관계를 맺지 않았고, Jpa도 사용하지 않았어서 해당 오류를 겪을 기회가 없었다. 실제 쿼리문 생성 시 fk관계를 맺지 않은 상태로 entity class에서 jpa 관계를 걸자 이런 오류가 발생한 것. 원인은 여러가지가 있겠으나 jpt 1차 캐시 또는 Lazing Loading 이슈, A테이블과 B테이블이 별도 쿼리로 조회됨에 따른 '시점차이'라는 ai의 답변이 있었음. 그러나 개인 로컬환경에서 개발 후 테스트 중이었기에

## 해결
as-is
```
//TableA(부모) : TableB(자식) = 1:N 매칭관계

//Table A Entity
//암것도 없음

//Table B Entity
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "A_id")
private TableA tableA;

//문제가 발생한 TableB repository(jpa repository 상속형태)
Optional<TableB> findById(Long id);

```

to-be
```
```