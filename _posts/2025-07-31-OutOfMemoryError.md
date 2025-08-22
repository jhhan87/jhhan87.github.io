---
title: java.lang.OutOfMemoryError- Java heap space error
description: out of memory
author: jhhan
date: 2025-07-31 17:50:00 +0900
categories: [Blogging, ErrorNote]
tags: [java, spring boot]
math: true
mermaid: true
---

# 발생일시
2024년 하반기의 어느 날...

# 발단
아파치 카프카를 써야하는 상황에서 카프카를 쓰기 어려워 대신 redis stream을 사용하기로 결정.
redis를 인증/인가를 위한 사용자 토큰관리용도로만 써봐서 redis stream을 처음 쓰는 과정에서 자바 out of memory error 발생

# 원인
정확히는 redis stream에 발행하는 로직을 잘못 구현하여 발행로직이 무한루프로 타기 시작하면서 몇백만건의 pending message가 쌓이고 있던 것

# 해결
