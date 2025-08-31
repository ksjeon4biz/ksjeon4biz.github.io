---
layout: post
title:  "Stateless와 RESTful, 제대로 이해하기"
date:   2025-08-31 16:13:00 +0900
categories: MSA
---

# Stateless와 RESTful, 제대로 이해하기
## 핵심 정의

* __Stateless(무상태):__ 서버가 클라이언트별 대화 상태를 보관하지 않는다는 뜻. 각 요청은 처리하는 데 필요한 모든 정보(인증, 리소스 식별, 행위 등)를 스스로 담아야 한다.

* __RESTful:__ Roy Fielding이 제시한 REST 제약조건(클라이언트-서버, 무상태, 캐시 가능, 계층화, 균일 인터페이스, 코드 온 디맨드[선택])을 HTTP 위에서 일관되게 적용해서, 리소스 중심으로 설계된 API 스타일을 말한다.

## Stateless를 조금 더 깊게

* __서버 세션 X:__ sessionId로 서버 메모리에 상태를 저장하고 찾는 패턴은 stateless 아님.

* __요청 자기완결성:__ 매 요청이 인증 토큰, 행위(HTTP 메서드), 대상(URI), 전제조건(ETag 등)을 모두 포함.

* ### 상태는 어디에?

* __리소스 상태:__ DB에 저장되는 데이터(서버의 “리소스 상태”).

* __애플리케이션 상태:__ 사용자의 진행 흐름(장바구니 단계, 페이지네이션 커서 등)은 클라이언트가 보유(쿼리스트링/바디/쿠키/JWT/링크).

* __장점:__ 수평 확장 쉬움(로드밸런싱), 장애격리, 캐싱 용이.

* __주의:__ 매 요청에 인증/맥락을 포함하므로 네트워크 비용과 보안(토큰 수명·회수)이 중요.

## RESTful의 구성요소(HTTP 활용 규칙)

### 1. 리소스 모델링

- 리소스는 명사형으로 식별: /users/42, /orders/2024-001, /products?category=books

- 행위는 HTTP 메서드로 표현:

    - GET 조회(안전)

    - POST 생성/서브리소스 행위

    - PUT 전체 치환(멱등)

    - PATCH 부분 업데이트(부분 변경)

    - DELETE 삭제(멱등적 의미를 갖도록 설계 권장)

### 2. 표현(Representation)

* JSON이 일반적. Content-Type: application/json, Accept: application/json

### 3. HTTP 상태코드

* 200 OK 일반 응답

* 201 Created + Location 헤더(신규 리소스)

* 204 No Content(본문 없음)

* 400/401/403/404/409/422/429/5xx 등 상황에 맞게

### 4. 하이퍼미디어(HATEOAS, 선택/권장)

* 응답에 다음 행동으로 가는 링크(예: "next": "/orders?page=3"), 전이(transition) 정보를 담아 클라이언트가 흐름을 스스로 탐색.

## 캐싱과 조건부 요청(성능·일관성)

* 캐시 가능성: Cache-Control, ETag/Last-Modified 헤더로 프록시/클라이언트 캐시 활용.

* 조건부 갱신(경쟁 상태 방지):

    * 읽기: 서버가 ETag: "v1abc" 반환

    * 업데이트: If-Match: "v1abc" 헤더와 함께 PUT/PATCH → 태그 불일치 시 412 Precondition Failed

* 조건부 조회: If-None-Match로 변경 없으면 304 Not Modified.

## 멱등성(Idempotency) & 재시도 안정성

* 멱등 메서드: 동일 요청을 여러 번 보내도 결과가 같음(GET, PUT, DELETE).

* POST의 멱등화: 결제·주문 등은 Idempotency-Key(요청 헤더/키)를 도입해 중복 생성 방지.

## 페이지네이션/정렬/필터링

* 쿼리 파라미터 사용:

    * /products?limit=20&cursor=eyJpZCI6... (커서 기반 권장)

    * /orders?from=2024-01-01&to=2024-12-31&status=PAID&sort=-created_at

* 응답 메타: next, prev 링크 또는 커서를 응답에 포함.

## 오류 응답(일관된 스키마)
```json
{
  "error": "validation_error",
  "message": "email is invalid",
  "details": { "email": ["must be a valid address"] },
  "trace_id": "req-7c1..."
}
```

* 사람이 읽을 메시지 + 머신이 파싱할 코드/필드를 같이.

## 버전 관리

* __URI 버전:__ /v1/orders (가장 흔함)

* __헤더 버전:__ Accept: application/vnd.example.orders+json;version=1

* 권장: 중요한 브레이킹 변경에만 버전 업, 가급적 호환 유지(필드 추가는 보통 브레이킹 아님).

## 보안/인증(Stateless와 잘 맞는 형태)

* __JWT Bearer (헤더 Authorization: Bearer <token>):__ 서버 세션 불필요 → 무상태.

* 수명 짧은 액세스 토큰 + 리프레시 토큰(재발급 엔드포인트는 별도로 보호).

* __권한(Authorization):__ 최소 권한 원칙, 리소스 소유자 확인, 감사 로그.

* __전달 최소화:__ 토큰은 필요할 때만 보내고, HTTPS 필수.

## 무엇이 RESTful이 아닌가

* 서버 세션에 기대는 로그인 흐름(세션 없는 요청은 진행 불가)

* 리소스 대신 동사형 RPC: /doCreateOrder

* 메서드 남용: 모든 작업을 POST /endpoint로만 처리

* 상태코드 무시: 항상 200 OK만 반환

### 예시: 주문 API 설계 스케치

```http
# 주문 생성
POST /orders
Content-Type: application/json
Idempotency-Key: 0f2c-...-a1

{ "user_id": "u_123", "items": [{"sku":"A1","qty":2}] }

201 Created
Location: /orders/2024-001
ETag: "v1xyz"
{
  "id": "2024-001",
  "status": "PENDING",
  "links": { "pay": "/orders/2024-001/payment" }
}

# 주문 상세 조회(조건부)
GET /orders/2024-001
If-None-Match: "v1xyz"

304 Not Modified

# 주문 부분 변경(경쟁 방지)
PATCH /orders/2024-001
If-Match: "v1xyz"
{ "shipping_address": { ... } }

200 OK
ETag: "v1xza"
```

## FlutterFlow + Supabase 맥락에서의 적용 팁

* Supabase 자동 REST(PostgREST)

    * 테이블/뷰가 곧 리소스 /rest/v1/<table> 형태.

    * JWT + RLS로 무상태 인증/인가 구현: 각 요청에 Authorization: Bearer <jwt>.

    * 필터/정렬: select=...&order=created_at.desc&limit=...&offset=... 또는 keyset(커서) 전환.

* Edge Functions(Deno)

    * 복잡한 유스케이스(결제·서드파티 연동)는 Edge Function을 리소스/행위 단위로 래핑.

    * Idempotency-Key, ETag/If-Match, 적절한 상태코드를 함수에서 명시.

* 트랜잭션/RPC

    * 다중 변경은 **DB 함수(RPC)**로 원자 처리 → 클라이언트는 짧고 단순한 요청 1회.

        - 예: 장바구니 수량 증감/체크아웃을 RPC로 캡슐화.

## 빠른 점검 체크리스트

* 서버가 세션 상태(메모리/스토어)에 의존하지 않는다(요청이 자기완결).

* 리소스는 명사형 URI, 행위는 HTTP 메서드로 표현한다.

* 상태코드/헤더(ETag, Location, Cache-Control)를 올바르게 사용한다.

* 페이지네이션/정렬/필터는 쿼리 파라미터와 링크로 드러낸다.

* 멱등성 보장: PUT/DELETE는 본질적으로, POST는 Idempotency-Key로.

* 인증은 JWT 등 무상태 방식, 권한은 RLS/정책으로 수립한다.

* 중복/레이턴시를 줄이기 위해 캐시·조건부 요청을 고려한다.