---
layout: post
title:  "Idempotent(멱등성)"
date:   2024-08-11 17:09:00 +0900
categories: architecture
---

Idempotent(멱등성)이라는 개념은 주로 컴퓨터 과학, 특히 웹 서비스와 API 설계에서 사용되며, 특정 연산이나 요청이 여러 번 반복되어 수행되더라도 결과가 변하지 않는 속성을 말합니다. 즉, 동일한 연산을 여러 번 적용하더라도 최종 결과는 처음 한 번 실행한 것과 동일합니다.

## Idempotent의 주요 특징
* 결과의 일관성: Idempotent 연산을 여러 번 실행해도 결과는 항상 동일합니다.
* 에러 처리 용이성: 네트워크 통신이나 시스템 오류로 인해 동일한 요청이 중복해서 전달되더라도, 시스템이 예상하지 못한 동작을 하지 않도록 보장합니다.

## Idempotent 예시
1. HTTP 메서드에서의 Idempotent
    HTTP/1.1 표준에서 특정 HTTP 메서드는 idempotent로 간주됩니다. 다음은 주요 HTTP 메서드와 그 idempotent 여부입니다.

    * GET: idempotent입니다. 예를 들어, GET /items/123 요청을 여러 번 보내도 동일한 결과(아이템 123의 정보)가 반환됩니다.
    * PUT: idempotent입니다. PUT /items/123 요청을 통해 특정 자원을 생성하거나 업데이트할 때, 동일한 요청을 여러 번 보내도 결과는 동일합니다.
    * DELETE: idempotent입니다. DELETE /items/123 요청을 여러 번 보내도, 첫 번째 요청에서 해당 자원이 삭제되었으므로 이후의 요청은 같은 결과(자원이 존재하지 않음)를 반환합니다.
    * POST: 일반적으로 idempotent가 아닙니다. POST /items 요청을 여러 번 보내면, 동일한 데이터가 여러 번 생성될 수 있습니다.

1. 데이터베이스 연산에서의 Idempotent
    * UPDATE: 특정 필드를 고정된 값으로 업데이트하는 연산은 idempotent입니다. 예를 들어, UPDATE users SET age = 30 WHERE id = 1라는 쿼리를 여러 번 실행해도 id = 1인 사용자의 나이는 30으로 유지됩니다.
    * INSERT: 일반적으로 idempotent가 아닙니다. 동일한 데이터를 여러 번 삽입하면 중복된 데이터가 생길 수 있습니다.

## Idempotent의 중요성
* 안전성: 네트워크 통신에서 요청이 중복될 수 있는 상황을 대비하여 시스템의 안정성을 높입니다.
* 복구 용이성: 시스템 장애 후 복구 과정에서 동일한 연산이 다시 실행되어도 시스템 상태에 영향을 미치지 않도록 보장합니다.
* 캐싱: idempotent한 요청은 캐싱이 용이하여, 성능 최적화에 유리합니다.

## Idempotent와 멱등 연산의 차이점
* 멱등 연산은 수학적으로는 어떤 값을 여러 번 적용해도 결과가 처음과 동일한 연산을 의미합니다. 예를 들어, 함수 f(x)가 f(f(x)) = f(x)를 만족할 때, 이 함수는 멱등성을 가집니다.
* Idempotent는 컴퓨터 과학에서 멱등성을 구현한 개념으로, 주로 시스템 설계에서 요청의 중복 처리에 관한 내용을 다룹니다.

## Idempotent의 예시 코드 (REST API)
```python
import requests

# Example of idempotent PUT request
url = "http://example.com/api/resource/123"
data = {"name": "Alice", "age": 30}

# Multiple PUT requests with the same data
response1 = requests.put(url, json=data)
response2 = requests.put(url, json=data)

# Both responses will result in the same outcome
print(response1.status_code)  # 200 OK
print(response2.status_code)  # 200 OK
```
이 코드에서 PUT 요청은 idempotent하기 때문에, 여러 번 실행해도 동일한 결과를 반환합니다.

## 요약
Idempotent는 연산이 여러 번 수행되더라도 시스템의 상태가 처음 한 번 수행했을 때와 동일하게 유지되는 성질을 의미합니다. 이를 통해 시스템은 중복 요청이나 오류로 인한 반복 실행에 대해 안정적으로 동작할 수 있습니다.


멱등성을 보장하기 위해서는 개발자가 해당 개념을 의도적으로 구현해야 합니다. 멱등성은 자동으로 보장되는 속성이 아니기 때문에, 개발자가 요청이나 연산이 여러 번 수행되더라도 동일한 결과가 나오도록 신경 써서 구현해야 합니다.

## 멱등성을 보장하기 위한 개발자의 역할
### 1. 데이터베이스 설계:

* 중복 데이터 방지: 예를 들어, 동일한 데이터를 여러 번 삽입하지 않도록 유니크 제약 조건을 설정하거나, 중복 삽입 요청이 들어왔을 때 무시하거나 업데이트하는 로직을 구현해야 합니다.
* 고정된 값 업데이트: UPDATE 쿼리에서 특정 필드를 고정된 값으로 설정하면 멱등성을 쉽게 구현할 수 있습니다. 예를 들어, SET age = 30은 여러 번 실행해도 항상 같은 결과를 제공합니다.

### 2. API 설계:

* HTTP 메서드 선택: REST API에서 멱등성을 보장하려면 올바른 HTTP 메서드를 선택해야 합니다. 예를 들어, 리소스를 업데이트할 때 PUT을 사용하면 멱등성을 보장할 수 있습니다. 반면, POST는 일반적으로 멱등성이 보장되지 않으므로 주의가 필요합니다.
* 중복 요청 처리: 중복된 요청이 들어왔을 때, 서버가 어떻게 처리할지를 명확히 정의해야 합니다. 예를 들어, 동일한 데이터로 여러 번 요청이 들어오면 첫 번째 요청 이후에는 아무 동작도 하지 않도록 구현할 수 있습니다.

### 3. 트랜잭션 관리:

* 트랜잭션 무결성: 트랜잭션이 실패하거나 중단될 경우, 동일한 트랜잭션이 다시 시도되더라도 동일한 결과를 보장하도록 트랜잭션 관리 시스템을 설계해야 합니다.
* 실패 재시도 로직: 네트워크 오류나 시스템 장애로 인해 동일한 요청이 여러 번 전달될 수 있으므로, 이러한 상황에 대비한 재시도 로직을 추가하고 멱등성을 유지할 수 있도록 해야 합니다.

### 4. 비즈니스 로직:

* Idempotent 연산 설계: 예를 들어, 금액을 추가로 더하는 연산보다는 특정 상태로 설정하는 연산이 멱등성을 유지하기 쉬운 경우가 많습니다.
* 요청 추적 및 로그 관리: 동일한 요청이 반복되었는지 여부를 추적할 수 있는 메커니즘을 도입하여 중복 처리로 인한 부작용을 방지할 수 있습니다.

## 예시: 멱등성 보장 방법
중복된 요청 무시

```java
public Response processRequest(String requestId, Data data) {
    if (isAlreadyProcessed(requestId)) {
        return new Response("Already Processed", HttpStatus.OK);
    }
    // 수행하고자 하는 작업 (예: 데이터베이스 업데이트)
    updateDatabase(data);
    markAsProcessed(requestId);
    return new Response("Success", HttpStatus.OK);
}

private boolean isAlreadyProcessed(String requestId) {
    // requestId가 이미 처리되었는지 확인하는 로직
}

private void markAsProcessed(String requestId) {
    // requestId를 처리된 상태로 기록
}
```

고정된 값으로 업데이트

```java
public Response updateUserAge(String userId, int age) {
    userRepository.updateAge(userId, age);  // age를 고정된 값으로 설정
    return new Response("Age updated", HttpStatus.OK);
}
```

## 요약
멱등성은 시스템의 안정성과 일관성을 보장하기 위해 중요한 개념입니다. 이를 구현하려면 개발자가 API 설계, 데이터베이스 처리, 트랜잭션 관리 등 여러 측면에서 멱등성을 고려하여 코드를 작성해야 합니다. 적절한 설계를 통해 멱등성을 보장하면, 중복된 요청이나 시스템 오류가 발생하더라도 시스템의 신뢰성을 높일 수 있습니다.
