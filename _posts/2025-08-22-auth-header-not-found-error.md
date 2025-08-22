---
title: auth header not found
description: Authorization Bearer header token not found
author: jhhan
date: 2025-08-22 12:50:00 +0900
categories: [Blogging, ErrorNote]
tags: [java, spring boot, http]
math: true
mermaid: true
---

## 발생일시
2025년 8월 22일

## 발단
프론트 개발자 한분이 수많은 api 중 jwt 토큰이 필요한 api 호출 시 종종 오류가 발생한다는 문의를 하심. 로직상으로는 문제가 없어보이고, 다른 api 호출이나 내가 테스트할때는 문제가 없었던지라 전체 플로우 확인 시작.
msa로 기능별 서버가 다 쪼개져있는 상황인데, 게이트웨이 서버와 인증서버 사이간 통신 시 프록시에서 인증헤더 누락을 의심하였고 처음에는 그쪽을 살펴봄.


## 원인
프록시 문제는 아니었음. 테스트 시 아이폰 사파리에서 웹뷰 접근하였는데, 테스트폰의 특정 조건에서 인증용헤더가 Authorization: Bearer..로 들어오는 것이 아니라 소문자(authorization: Bearer..) 로 자동변환되어 들어왔음. 인증서버에서 인증헤더 조회 시 단순 equal처리 되어 있었어서 발생한 문제였음.
claude에게 물어보니 http/2 사용 시 http헤더를 소문자로 변환한다고 함 (http/1.1에서는 상관 없었으나 2부터 명시적 규정)
이미 오래전에 표준으로 지정된 내용인데 이제 발견되었다는게 놀라움. 대다수의 요청이 http2 아닌 1.1로 요청이 오기 때문에 그랬던 것으로 예상됨.


## 해결
as-is
```
//BEARER_PREFIX : "Bearer ";
//AUTHORIZATION_HEADER : "Authorization";
        String authHeader = request.getHeaders().getFirst(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(authHeader) && authHeader.startsWith(BEARER_PREFIX)) {
            return authHeader.substring(BEARER_PREFIX.length()).trim();
        }
```

to-be
```
        Optional<String> authHeader = request.getHeaders().entrySet().stream()
                .filter(entry -> entry.getKey().equalsIgnoreCase(AUTHORIZATION_HEADER))
                .map(Map.Entry::getValue)
                .filter(values -> !values.isEmpty())
                .map(List::getFirst)
                .findFirst();

        String headerValue = authHeader.get();
        if (StringUtils.hasText(headerValue) && headerValue.startsWith(BEARER_PREFIX)) {
            return headerValue.substring(7).trim();
        }
```