---
layout: post
title:  "SAGA"
date:   2024-08-11 17:33:00 +0900
categories: pattern
---

SAGA 패턴은 마이크로서비스 아키텍처(MSA)에서 분산 트랜잭션을 관리하기 위한 패턴 중 하나입니다. 마이크로서비스 환경에서는 여러 서비스가 독립적으로 운영되기 때문에, 전통적인 데이터베이스 트랜잭션(ACID)을 적용하기 어렵습니다. SAGA 패턴은 이러한 환경에서 데이터 일관성을 유지하면서도 서비스 간의 연속적인 작업을 조율할 수 있는 방법을 제공합니다.

## SAGA 패턴의 개념
SAGA 패턴은 하나의 큰 트랜잭션을 여러 개의 작은 단위 작업(transaction)으로 나누어 관리합니다. 각 단위 작업은 하나의 마이크로서비스에서 수행되며, 모든 단위 작업이 성공적으로 완료되면 전체 트랜잭션이 완료된 것으로 간주됩니다. 만약 어떤 단위 작업에서 오류가 발생하면, 이미 완료된 단위 작업을 보상 작업(compensating transaction)을 통해 롤백하거나 수정하여 데이터 일관성을 유지합니다.

## SAGA 패턴의 두 가지 주요 구현 방식
### 1. Choreography(오케스트레이션 없는 패턴):

* 각 서비스는 다음 작업을 수행할 서비스에 이벤트를 전송하여 트랜잭션을 이어가는 방식입니다.
* 중앙에서 조율하는 관리자가 없고, 서비스들이 서로 직접 통신하며 다음 작업을 알아서 실행합니다.
* 장점: 간단한 설계, 추가적인 중앙 관리 서비스 필요 없음.
* 단점: 서비스 간 결합도가 증가할 수 있으며, 복잡한 트랜잭션의 경우 관리가 어려울 수 있음.

### 2. Orchestration(오케스트레이션 패턴):

* 중앙 조정자(orchestrator)가 전체 SAGA를 관리합니다. 이 조정자는 각 서비스에 작업을 요청하고, 작업 완료 여부를 확인하며, 필요시 보상 작업을 지시합니다.
* 모든 트랜잭션 흐름은 중앙 조정자가 관리하며, 서비스들은 요청을 받아 수행합니다.
* 장점: 트랜잭션의 흐름을 중앙에서 관리하기 때문에 복잡한 트랜잭션도 체계적으로 관리 가능.
* 단점: 중앙 조정자가 단일 장애점(single point of failure)이 될 수 있으며, 복잡성이 증가할 수 있음.

## SAGA 패턴의 예시
예를 들어, 전자 상거래 애플리케이션에서 고객이 주문을 할 때의 프로세스를 생각해 봅시다:

1. 주문 생성 서비스: 고객의 주문을 생성하고 데이터베이스에 저장합니다.
1. 결제 서비스: 결제를 처리합니다.
1. 재고 관리 서비스: 주문한 상품이 있는지 확인하고, 재고를 줄입니다.
1. 배송 서비스: 상품을 고객에게 발송합니다.

만약 재고 관리 서비스에서 문제가 발생하여 재고가 부족한 경우, SAGA 패턴을 사용해 이전의 작업들을 롤백하는 보상 작업을 수행할 수 있습니다. 예를 들어, 결제를 취소하고, 주문 데이터를 삭제하는 등의 작업을 수행할 수 있습니다.

## SAGA 패턴의 장점
* 데이터 일관성 유지: 분산된 환경에서도 데이터 일관성을 유지할 수 있습니다.
* 트랜잭션 실패 관리: 작업이 실패하면 보상 작업을 통해 복구할 수 있습니다.
* 확장성: 각 서비스가 독립적으로 작업을 수행하므로, 시스템의 확장성이 높습니다.

## SAGA 패턴의 단점
* 복잡성: 트랜잭션이 여러 서비스에 걸쳐 있을 경우, 복잡성이 증가할 수 있습니다.
* 지연: 각 단위 작업이 순차적으로 수행되므로 전체 트랜잭션의 완료 시간이 느릴 수 있습니다.
* 보상 작업의 어려움: 보상 작업을 설계하는 것이 복잡하고 어려울 수 있습니다.

## 결론
SAGA 패턴은 마이크로서비스 아키텍처에서 분산 트랜잭션을 효과적으로 관리하는 데 유용한 패턴입니다. 이 패턴을 통해 데이터 일관성을 유지하면서도 유연하고 확장 가능한 시스템을 설계할 수 있습니다. 다만, 설계와 구현 단계에서 복잡성을 잘 관리해야 하는 도전 과제가 따릅니다.


- - - 


SAGA 패턴에서 후행 서비스가 실패하고 보상 트랜잭션이 이루어지기 전에 선행 서비스를 조회하는 경우, 조회하는 시점에 따라 두 가지 시나리오가 발생할 수 있습니다.

### 1. 보상 트랜잭션 전 조회: 일관성 문제
* 만약 후행 서비스가 실패하고 보상 트랜잭션이 실행되기 전에 선행 서비스의 상태를 조회하면, 해당 서비스는 아직 보상 트랜잭션이 수행되지 않았기 때문에 일시적으로 일관성 없는 상태를 반환할 수 있습니다.
* 예를 들어, 항공권 예약 시스템에서 SAGA 패턴을 사용하는 경우를 생각해봅시다. 사용자가 항공권을 예약한 후 호텔 예약 단계에서 실패했다면, 항공권 예약을 취소하는 보상 트랜잭션이 실행되어야 합니다. 그러나 이 보상 트랜잭션이 실행되기 전에 항공권 상태를 조회하면, 항공권이 여전히 예약된 상태로 보일 수 있습니다.

### 2. 보상 트랜잭션 후 조회: 일관성 유지
* 보상 트랜잭션이 성공적으로 완료된 후에 선행 서비스를 조회하면, 일관된 상태가 유지됩니다. 예를 들어, 보상 트랜잭션으로 인해 항공권 예약이 취소되었다면, 그 이후에 항공권 상태를 조회하면 예약이 취소된 상태로 조회됩니다.

## SAGA 패턴의 일관성 관리
SAGA 패턴은 일반적으로 **사후 일관성(Eventual Consistency)**을 보장합니다. 즉, 각 단계가 개별적으로 수행되며, 최종적으로 모든 트랜잭션이 성공하거나 보상 트랜잭션이 완료된 후에는 시스템이 일관된 상태에 도달합니다. 그러나 트랜잭션이 진행 중이거나 보상 트랜잭션이 아직 완료되지 않은 시점에는 잠시 동안 일관성 없는 상태가 발생할 수 있습니다.

이를 방지하거나 최소화하기 위해 다음과 같은 기법을 사용할 수 있습니다:

### 1. 상태 플래그 사용:

* 서비스 상태에 플래그를 두어, 트랜잭션이 완전히 완료되지 않았음을 나타냅니다. 예를 들어, "예약 중" 또는 "취소 중"과 같은 상태를 사용하여 아직 최종 상태에 도달하지 않았음을 명확히 표시할 수 있습니다.

### 2. 읽기 차단(Read Blocking):

* 특정 조건에서 읽기 요청을 일시적으로 차단하거나, 읽기 요청이 있을 경우 최종 상태를 확인할 때까지 대기하게 할 수 있습니다. 이를 통해 트랜잭션이 완료될 때까지 일관성 없는 데이터를 노출하지 않도록 합니다.

### 3. 읽기 모델의 분리(CQRS 패턴):

* Command Query Responsibility Segregation (CQRS) 패턴을 사용하여 쓰기와 읽기 모델을 분리할 수 있습니다. SAGA 패턴을 사용하는 트랜잭션에서 상태가 확정되기 전에는 읽기 모델이 일시적으로 일관성 없는 상태를 반환하지 않도록 설계할 수 있습니다.

## 요약
SAGA 패턴에서 후행 서비스가 실패하고 보상 트랜잭션이 이루어지기 전에 선행 서비스의 상태를 조회하면 일관성 없는 상태를 볼 수 있습니다. 이를 해결하기 위해 상태 플래그, 읽기 차단, CQRS 패턴 등을 활용하여 일관성을 관리할 수 있습니다. SAGA 패턴은 사후 일관성을 목표로 하기 때문에, 최종적으로는 모든 서비스의 상태가 일관되게 유지됩니다.


---


보상 작업(Compensating Transaction)은 SAGA 패턴에서 중요한 개념으로, 분산 트랜잭션의 일부가 실패했을 때 이미 완료된 이전 작업의 결과를 취소하거나 수정하기 위해 수행하는 작업을 의미합니다. 이는 전통적인 데이터베이스 트랜잭션에서의 롤백(rollback) 개념과 유사하지만, 분산 시스템의 특성상 각 서비스가 독립적으로 운영되기 때문에 롤백이 중앙에서 자동으로 이루어지지 않습니다. 따라서 각 서비스는 자체적으로 보상 작업을 정의하고 수행해야 합니다.

## 보상 작업의 원리
보상 작업은 일반적으로 다음과 같은 원리로 설계되고 수행됩니다:

1. __원자성 유지__: 분산된 환경에서 ACID 트랜잭션의 원자성을 보장할 수 없으므로, SAGA 패턴은 각 작업이 독립적으로 수행될 수 있도록 설계합니다. 만약 어떤 단계에서 오류가 발생하면, 그 단계 이전에 수행된 작업들을 취소하기 위해 보상 작업이 호출됩니다.

1. __선형적인 실행__: SAGA에서의 트랜잭션은 순차적으로 실행됩니다. 예를 들어, A → B → C 순으로 작업이 진행되고, C 단계에서 실패하면, B의 결과를 취소하기 위해 보상 작업 B'가 실행되고, 그 다음 A의 결과를 취소하기 위해 보상 작업 A'가 실행됩니다.

1. __비대칭성__: 보상 작업은 원래 작업과 동일한 방식으로 수행되지 않습니다. 보상 작업은 데이터를 삭제하거나 이전 상태로 되돌리기 위해 특별히 설계된 작업입니다. 예를 들어, 원래 작업이 데이터베이스에 레코드를 삽입하는 것이었다면, 보상 작업은 해당 레코드를 삭제하는 작업이 될 수 있습니다.

## 보상 작업의 예시
전자 상거래 시스템에서 고객이 주문을 처리하는 과정에서 발생할 수 있는 보상 작업의 예를 들어 보겠습니다:

1. ### 주문 생성 서비스:

    * 원래 작업: 고객의 주문을 생성하고 데이터베이스에 저장합니다.
    * 보상 작업: 주문을 데이터베이스에서 삭제합니다.

1. ### 결제 서비스:

    * 원래 작업: 고객의 결제를 처리하고 승인합니다.
    * 보상 작업: 결제를 취소하고 고객의 계좌에 금액을 환불합니다.

1. ### 재고 관리 서비스:

    * 원래 작업: 주문한 상품에 대해 재고를 줄입니다.
    * 보상 작업: 재고를 다시 추가하여 원래 상태로 복원합니다.

1. ### 배송 서비스:

    * 원래 작업: 상품을 고객에게 배송합니다.
    * 보상 작업: 배송을 취소하고, 상품을 창고로 반품합니다.

이 예시에서, 만약 배송 서비스에서 상품을 배송할 수 없는 문제가 발생했다면, SAGA 패턴은 다음과 같이 동작할 수 있습니다:

1. 배송을 취소하는 보상 작업이 먼저 수행됩니다.
1. 이어서 재고 관리 서비스에서 이미 차감된 재고를 다시 추가합니다.
1. 마지막으로 결제 서비스에서 고객에게 환불을 처리합니다.

## 보상 작업 설계 시 고려 사항
보상 작업을 설계할 때는 다음과 같은 사항들을 고려해야 합니다:

1. __작업의 불변성__: 원래 작업이 성공한 후에는 그 결과가 다른 트랜잭션에서 참조되거나 의존할 수 있기 때문에, 보상 작업을 수행할 때 기존 데이터를 안전하게 취소할 수 있는지 확인해야 합니다.

1. __복잡한 데이터 상태 관리__: 보상 작업은 데이터 상태를 다시 이전 상태로 되돌리는 작업을 수행해야 하므로, 데이터가 복잡한 경우 이를 제대로 복원할 수 있는지 고려해야 합니다.

1. __타이밍 문제__: 분산 시스템에서는 네트워크 지연이나 서비스 중단 등으로 인해 보상 작업이 제때 수행되지 않을 수 있습니다. 이를 대비하여, 보상 작업이 지연되더라도 시스템이 안정적으로 작동할 수 있도록 설계해야 합니다.

1. __아이템포턴시(Idempotency)__: 보상 작업은 아이템포턴트해야 합니다. 즉, 동일한 보상 작업이 여러 번 호출되더라도 시스템 상태가 변하지 않도록 해야 합니다. 이는 네트워크 재시도 시 오류를 방지하는 데 중요합니다.

복잡한 트랜잭션 시나리오: 일부 트랜잭션 시나리오는 보상 작업 설계가 복잡할 수 있습니다. 예를 들어, 부분적으로 성공한 작업이 여러 서비스에 영향을 미치는 경우, 이러한 복잡성을 관리하기 위한 별도의 로직이나 상태 추적이 필요할 수 있습니다.

## 보상 작업의 장점과 단점
### 장점:
* __데이터 일관성__: 분산 시스템에서의 데이터 일관성을 유지하는 데 중요한 역할을 합니다.
* __유연성__: 각 서비스가 독립적으로 보상 작업을 정의하고 수행할 수 있어, 시스템 전체의 유연성이 높아집니다.
* __확장성__: 보상 작업을 통해 시스템의 확장성과 복원력을 향상시킬 수 있습니다.

### 단점:
* __설계 복잡성__: 보상 작업을 설계하고 구현하는 것은 복잡할 수 있습니다. 특히 여러 서비스에 걸쳐 있는 트랜잭션의 경우 더 많은 고려 사항이 필요합니다.
* __일관성 관리__: 보상 작업이 적절하게 수행되지 않으면 데이터 일관성이 깨질 수 있으며, 이는 시스템의 안정성에 큰 영향을 미칠 수 있습니다.

## 결론
보상 작업은 SAGA 패턴에서 분산 트랜잭션의 복잡성을 관리하기 위해 필수적인 요소입니다. 올바르게 설계된 보상 작업은 시스템의 안정성을 유지하고, 장애 발생 시에도 데이터 일관성을 유지할 수 있게 해줍니다. 그러나 이를 구현할 때는 시스템의 특성, 데이터 상태, 서비스 간의 의존성 등을 면밀히 분석하고 설계해야 합니다.
