---
layout: post
title:  "Cloud MSA Azure"
date:   2024-08-11 19:18:00 +0900
categories: architecture
---

Azure Cloud에서 위 요구사항을 기반으로 마이크로서비스 아키텍처(MSA)를 설계하는 방법을 제안하겠습니다. 이 설계는 온라인 상품 주문을 처리하는 시스템과 실시간 배송 추적 시스템을 포함합니다. RDBMS와 NoSQL 데이터베이스를 사용하는 요구사항, 그리고 주문량의 변동성을 고려하여 확장 가능하고 유연한 아키텍처를 구성하겠습니다.

## 아키텍처 개요
1. __API Gateway__: 모든 클라이언트 요청을 라우팅하고 인증 및 인가를 처리합니다.

1. __주문 서비스(Order Service)__: 상품 주문을 처리하며, 주문 정보를 RDBMS에 저장합니다.

1. __배송 추적 서비스(Delivery Tracking Service)__: 실시간 배송 추적 정보를 관리하며, NoSQL 데이터베이스에 저장합니다.

1. __이벤트 버스(Event Bus)__: 서비스 간의 비동기 통신을 지원합니다.

1. __모니터링 및 로깅__: 시스템 상태와 성능을 모니터링하고 로그를 수집합니다.

1. __자동 확장(Auto Scaling)__: 주문량의 변동에 따라 서비스 인스턴스를 자동으로 조정합니다.

## 주요 구성 요소
1. ### API Gateway
    * __Azure API Management__: 클라이언트 요청을 각 서비스로 라우팅하고, 인증 및 인가를 처리합니다.

    * __역할__: 요청 라우팅, 인증 및 인가, 로깅, 모니터링, 제한(Throttle) 등.

1. ### 주문 서비스 (Order Service)
    * __Azure Kubernetes Service (AKS)__: 주문 서비스의 컨테이너화된 마이크로서비스를 실행합니다.

    * __Azure SQL Database__: 주문 정보를 저장하는 RDBMS로 사용합니다.

    * __주문량 증가 대응__: 주말 주문량 증가에 대비하여 AKS의 자동 확장을 설정합니다.

    * __서비스 구성__: 주문 생성, 조회, 업데이트, 삭제 API를 제공합니다.

1. ### 배송 추적 서비스 (Delivery Tracking Service)
    * __Azure Kubernetes Service (AKS)__: 배송 추적 서비스의 컨테이너화된 마이크로서비스를 실행합니다.

    * __Azure Cosmos DB__: 실시간 배송 추적 정보를 저장하는 NoSQL 데이터베이스로 사용합니다.

    * __서비스 구성__: 배송 상태 업데이트, 조회 API를 제공합니다.

1. ### 이벤트 버스 (Event Bus)
    * __Azure Service Bus 또는 Azure Event Grid__: 서비스 간 비동기 메시지 전송을 위해 사용합니다.

    * __역할__: 주문 생성 시 배송 추적 서비스로 이벤트 전송, 주문 상태 변경 알림 등.

1. ### 모니터링 및 로깅
    * __Azure Monitor__: 전체적인 시스템 모니터링과 로그 수집을 담당합니다.

    * __Azure Log Analytics__: 로그 데이터를 분석하고 쿼리합니다.

    * __Application Insights__: 각 서비스의 성능 모니터링 및 디버깅을 지원합니다.

1. ### 자동 확장 (Auto Scaling)
    * __Azure Kubernetes Service (AKS)__: CPU 사용률, 메모리 사용률 등의 메트릭을 기준으로 자동 확장을 설정합니다.

    * __Azure SQL Database__: 자동 스케일링을 통해 주문량 증가에 대응합니다.

    * __Cosmos DB__: 자동 확장 기능을 통해 트래픽 증가에 대응합니다.

## 설계 상세
1. ### API Gateway 설정
    * Azure API Management를 사용하여 API Gateway를 구성합니다.

    * 정책을 설정하여 인증, 인가, 제한 등을 관리합니다.

1. ### 주문 서비스 설계
    * __컨테이너화__: 주문 서비스를 Docker 컨테이너로 패키징하여 Azure Kubernetes Service(AKS)에 배포합니다.

    * __데이터베이스__: Azure SQL Database를 사용하여 주문 정보를 저장합니다. 기본적인 스키마는 고객 정보, 주문 정보, 상품 정보 등을 포함합니다.

    * __자동 확장__: AKS에서 Horizontal Pod Autoscaler를 설정하여 트래픽 증가 시 자동으로 주문 서비스의 인스턴스를 늘립니다.

1. ### 배송 추적 서비스 설계
    * __컨테이너화__: 배송 추적 서비스를 Docker 컨테이너로 패키징하여 AKS에 배포합니다.

    * __데이터베이스__: Azure Cosmos DB를 사용하여 실시간 배송 정보를 저장합니다. 스키마리스 데이터 구조를 활용하여 유연한 데이터 저장이 가능합니다.

    * __실시간 업데이트__: Cosmos DB의 Change Feed 기능을 활용하여 실시간 데이터 변경 사항을 처리합니다.

1. ### 이벤트 버스 설정
    * Azure Service Bus를 사용하여 서비스 간의 메시지 전달을 관리합니다.

    * __주문 생성 이벤트__: 주문 서비스에서 주문이 생성되면 이벤트를 발행하여 배송 추적 서비스가 이를 구독하고 처리합니다.

1. ### 모니터링 및 로깅 설정
    * Azure Monitor를 사용하여 전체 시스템의 성능을 모니터링합니다.

    * Application Insights를 각 마이크로서비스에 통합하여 상세한 애플리케이션 모니터링 및 로깅을 구현합니다.

    * __Log Analytics__: 모든 로그 데이터를 중앙에서 분석할 수 있도록 설정합니다.

1. ### 자동 확장 설정
    * AKS의 Horizontal Pod Autoscaler를 설정하여 CPU 및 메모리 사용률을 기반으로 자동 확장을 구성합니다.

    * Azure SQL Database 및 Cosmos DB의 자동 확장 기능을 활용하여 데이터베이스 레벨에서의 확장을 처리합니다.

## 결론
이 설계를 통해 MSA 기반의 온라인 주문 및 실시간 배송 추적 시스템을 Azure Cloud에서 효과적으로 구현할 수 있습니다. 각 서비스는 독립적으로 배포 및 확장 가능하며, Azure의 다양한 관리형 서비스를 활용하여 확장성과 유연성을 극대화할 수 있습니다. 특히 주말과 평일의 주문량 변동에 대응할 수 있도록 자동 확장을 설정하여 안정적인 서비스 운영을 보장합니다.


---
---


가용성을 고려할 때는 단순히 데이터베이스를 다른 리전에 두는 것만으로는 충분하지 않습니다. 전체 아키텍처에 걸쳐 다양한 요소를 다중 리전에 분산시켜야 합니다. 여기에는 애플리케이션, 데이터베이스, API Gateway, 이벤트 버스 및 기타 인프라 구성 요소가 포함됩니다. Azure에서 다중 리전 아키텍처를 설계할 때 고려해야 할 사항은 다음과 같습니다.

## 다중 리전 아키텍처 설계
1. ### 애플리케이션 배포
    * Azure Kubernetes Service (AKS): AKS 클러스터를 여러 리전에 배포하여 애플리케이션의 고가용성을 확보합니다. 각각의 리전에 독립적인 클러스터를 두고, 클러스터 간의 트래픽을 라우팅합니다.

1. ### 데이터베이스 배포
    * Azure SQL Database:

        * Active Geo-Replication: 주 데이터베이스를 한 리전에 두고, 다른 리전에 읽기 전용 복제본을 설정하여 재해 복구 및 읽기 성능 향상을 도모합니다.

        * Auto-Failover Group: 데이터베이스를 여러 리전에 걸쳐 자동 장애 조치(Failover)를 설정합니다.

    * Azure Cosmos DB:

        * 다중 지역 배포: Cosmos DB는 기본적으로 다중 지역에서 데이터 복제를 지원합니다. 데이터가 여러 리전에 걸쳐 분산되어 있어 높은 가용성과 성능을 제공합니다.

        * 자동 장애 조치: Cosmos DB는 지역 장애 시 자동으로 장애 조치를 수행합니다.

1. ### API Gateway
    * Azure Front Door: 여러 리전에 배포된 애플리케이션으로 트래픽을 분산시키고, 자동 장애 조치 및 글로벌 부하 분산을 제공합니다.

1. ### 이벤트 버스
    * Azure Service Bus 또는 Azure Event Grid:

        * Geo-Disaster Recovery: Azure Service Bus는 재해 복구 기능을 제공하여 메시지 큐를 여러 리전에 복제합니다.

        * Event Grid: 이벤트를 다중 리전에 걸쳐 게시하고 구독할 수 있도록 설정합니다.

1. ### DNS 관리
    * Azure Traffic Manager: 여러 리전에 배포된 서비스로 트래픽을 분산시키기 위해 DNS 라우팅을 제공합니다.

    * 라우팅 메서드: 성능 라우팅, 가중치 라우팅, 우선 순위 라우팅 등 다양한 라우팅 방법을 사용하여 트래픽을 최적화합니다.

## 재해 복구 전략
1. ### 데이터 복제 및 백업:
    * 데이터베이스의 정기적인 백업 및 여러 리전에 걸친 데이터 복제를 설정하여 데이터 손실을 방지합니다.

1. ### 자동 장애 조치(Failover):
    * 주요 서비스와 데이터베이스에 대해 자동 장애 조치 설정을 통해 특정 리전에 문제가 발생할 경우 자동으로 다른 리전으로 전환됩니다.

1. ### 헬스 체크 및 모니터링:
    * 모든 리전에서 서비스 상태를 모니터링하고, 문제가 발생하면 신속하게 조치할 수 있도록 헬스 체크를 설정합니다.

1. ### 테스트 및 시뮬레이션:
    * 정기적으로 재해 복구 시나리오를 테스트하고 시뮬레이션하여 모든 구성 요소가 예상대로 작동하는지 확인합니다.

## 예시 아키텍처
1. ### 주문 서비스(Order Service):
    * 주 리전: 미국 서부
    * 보조 리전: 미국 동부
    * Active Geo-Replication을 사용하여 Azure SQL Database를 복제하고, Azure Traffic Manager로 트래픽을 분산합니다.

1. ### 배송 추적 서비스(Delivery Tracking Service):
    * 주 리전: 미국 서부
    * 보조 리전: 미국 동부
    * 다중 지역 배포를 사용하여 Cosmos DB를 복제하고, Azure Front Door를 사용하여 글로벌 부하 분산을 설정합니다.

1. ### API Gateway:
    * Azure Front Door를 사용하여 여러 리전에 걸친 API 게이트웨이를 구성합니다.

1. ### 이벤트 버스:
    * Azure Service Bus를 사용하여 여러 리전에 걸친 메시지 큐를 구성하고 Geo-Disaster Recovery를 설정합니다.

이와 같이 설계하면 각 리전에 걸쳐 고가용성 및 재해 복구를 지원하는 다중 리전 아키텍처를 구현할 수 있습니다. 이를 통해 서비스 중단 시에도 안정적인 서비스 운영이 가능합니다.


---
---


주 리전에 있는 주문 서비스가 보조 리전에 있는 Azure SQL Database로 트래픽을 보내는 것은 일반적으로 지양해야 합니다. 주 리전과 보조 리전 간의 데이터 복제는 주 리전의 데이터베이스에서 보조 리전의 읽기 전용 복제본으로 일방향으로 이루어지기 때문입니다. 이로 인해 다음과 같은 문제들이 발생할 수 있습니다.

## 문제점
1. ### 데이터 일관성 문제:
    * 보조 리전의 데이터베이스는 읽기 전용 복제본입니다. 쓰기 작업이 지원되지 않기 때문에, 주문 서비스가 보조 리전의 데이터베이스로 쓰기 트래픽을 보낼 수 없습니다.

1. ### 지연 시간 증가:
    * 주 리전의 애플리케이션이 보조 리전의 데이터베이스로 접근할 경우, 리전 간 네트워크 지연이 발생하여 성능이 저하될 수 있습니다.

1. ### 복잡한 장애 복구:
    * 주 리전에서 보조 리전으로의 트래픽 라우팅은 복잡한 장애 복구 전략을 요구하며, 이는 시스템 복구 시 혼란을 초래할 수 있습니다.

## 올바른 접근 방법
1. __리전 간 데이터베이스 복제__: 주 리전의 데이터베이스는 쓰기 작업을 처리하고, 보조 리전의 데이터베이스는 읽기 작업을 처리할 수 있도록 Active Geo-Replication을 설정합니다.

1. __지역별 서비스 배포__: 주문 서비스를 주 리전과 보조 리전에 각각 독립적으로 배포하고, 트래픽을 각각의 지역 데이터베이스로 라우팅합니다.

## 개선된 설계
1. ### 주문 서비스
    * 주 리전: 미국 서부
        * AKS 클러스터: 주 리전에 배포된 주문 서비스
        * Azure SQL Database: 주 리전에서 쓰기 및 읽기 작업을 처리

    * 보조 리전: 미국 동부
        * AKS 클러스터: 보조 리전에 배포된 주문 서비스
        * Azure SQL Database: 보조 리전에서 읽기 전용 작업을 처리
        * 자동 장애 조치: 주 리전의 데이터베이스에 문제가 발생하면, 보조 리전의 데이터베이스로 자동 장애 조치

1. ### Azure Traffic Manager
    * 라우팅 메서드: 우선 순위 라우팅 또는 성능 라우팅을 사용하여 주 리전과 보조 리전 간의 트래픽을 분배합니다.

1. ### Azure Front Door
    * 글로벌 부하 분산: 주 리전과 보조 리전 간의 트래픽을 글로벌하게 분산하고, 장애 발생 시 자동으로 트래픽을 전환합니다.

## 장애 복구 시나리오
1. ### 주 리전의 장애 발생:
    * Azure Traffic Manager와 Azure Front Door는 자동으로 보조 리전으로 트래픽을 전환합니다.

    * 보조 리전의 주문 서비스가 활성화되고, 읽기 전용 데이터베이스에서 읽기 작업을 처리합니다.

    * 주 리전 복구 후: 트래픽을 다시 주 리전으로 전환하고, 데이터베이스 복제를 동기화합니다.

1. ### 보조 리전의 장애 발생:
    * 주 리전의 서비스와 데이터베이스는 계속 정상적으로 운영됩니다.

    * 보조 리전 복구 후 데이터베이스 복제를 다시 설정합니다.

## 결론
주 리전과 보조 리전에 걸쳐 고가용성을 확보하기 위해 각 리전에 독립적으로 서비스를 배포하고, 데이터베이스 복제를 통해 일관성을 유지하는 것이 중요합니다. Azure Traffic Manager와 Azure Front Door를 활용하여 리전 간의 트래픽을 효율적으로 분배하고, 자동 장애 조치를 통해 서비스 중단 시 신속하게 복구할 수 있습니다.


---
---


정리하면 주 리전에 장애가 발생하면 보조 리전에 쓰기가 가능해져야 합니다. 이를 위해서는 Azure SQL Database의 Auto-Failover Group 기능을 사용하여 자동으로 장애 조치(Failover)를 설정하는 것이 중요합니다. 아래에 그 과정을 더 자세히 설명하겠습니다.

## Auto-Failover Group 설정
Azure SQL Database의 Auto-Failover Group 기능을 사용하면 주 리전에서 보조 리전으로 데이터베이스를 자동으로 장애 조치할 수 있습니다. 이를 통해 주 리전이 장애가 발생하더라도 보조 리전에서 읽기 및 쓰기 작업이 가능해집니다.

### 설정 과정
1. #### 데이터베이스 복제 설정:
    * 주 리전의 Azure SQL Database를 보조 리전에 복제합니다.

    * 보조 리전의 데이터베이스는 기본적으로 읽기 전용입니다.

1. #### Auto-Failover Group 생성:
    * 주 리전과 보조 리전 간의 Auto-Failover Group을 생성합니다.

    * Auto-Failover Group은 주 리전의 기본 데이터베이스와 보조 리전의 읽기 전용 복제본을 포함합니다.

1. #### Failover 설정:
    * 장애 발생 시 자동으로 주 리전에서 보조 리전으로 장애 조치(Failover)하도록 설정합니다.

    * 장애 조치 후 보조 리전의 데이터베이스가 쓰기 가능 상태로 전환됩니다.

### 장애 발생 시나리오
1. #### 주 리전 정상 상태:
    * 주 리전: 쓰기 및 읽기 작업을 처리합니다.

    * 보조 리전: 읽기 전용 복제본으로 데이터베이스를 유지합니다.

1. #### 주 리전 장애 발생:
    * Auto-Failover Group이 자동으로 보조 리전으로 장애 조치를 수행합니다.

    * 보조 리전: 데이터베이스가 읽기/쓰기가 가능하도록 전환됩니다.

1. #### 장애 복구 후:
    * 주 리전 복구 후 Auto-Failover Group을 통해 주 리전 데이터베이스를 다시 주 데이터베이스로 설정하고 보조 리전 데이터베이스를 복제본으로 전환합니다.

### Azure 환경에서의 설정 예시
1. #### Azure SQL Database 복제 및 Auto-Failover Group 생성:
```shell
az sql db replica create --name primaryDB --resource-group myResourceGroup --server primaryServer --partner-server secondaryServer --partner-database secondaryDB --secondary-type Geo --failover-group myFailoverGroup
```

1. #### Failover Group을 통한 장애 조치 설정:
```shell
az sql failover-group create --name myFailoverGroup --resource-group myResourceGroup --server primaryServer --partner-server secondaryServer
az sql failover-group update --name myFailoverGroup --resource-group myResourceGroup --server primaryServer --add-db primaryDB
```

1. #### 주 리전 장애 발생 시 장애 조치 확인:
```shell
az sql failover-group failover --name myFailoverGroup --resource-group myResourceGroup --server secondaryServer
```

## 요약
* Auto-Failover Group을 사용하여 주 리전과 보조 리전 간에 자동 장애 조치를 설정합니다.
* 장애 발생 시 보조 리전의 데이터베이스가 자동으로 쓰기 가능 상태로 전환되어 서비스 연속성을 유지합니다.
* 주 리전 복구 후 주 리전으로 다시 장애 조치를 수행하여 원래 상태로 복구합니다.

이러한 설정을 통해 주 리전에 장애가 발생해도 보조 리전에서 쓰기가 가능하게 하여 고가용성을 보장할 수 있습니다.


---
---


보조 리전으로 들어온 쓰기 요청이 주 리전 데이터베이스로 라우팅되는 것은 올바르지 않습니다. 주 리전에 장애가 발생하여 보조 리전으로 장애 조치(failover)가 이루어졌을 때는, 보조 리전 데이터베이스가 쓰기 가능한 주 데이터베이스 역할을 해야 합니다. 장애 조치 후 주 리전이 복구되면 다시 주 리전 데이터베이스로 장애 조치를 수행할 수 있습니다.

### 장애 조치(failover) 과정 요약
1. #### 정상 운영 상태:
    * 주 리전: 읽기 및 쓰기 작업을 처리하는 주 데이터베이스.

    * 보조 리전: 읽기 전용 복제본.

1. #### 주 리전 장애 발생:
    * Auto-Failover Group이 자동으로 보조 리전으로 장애 조치(failover)합니다.

    * 보조 리전 데이터베이스: 쓰기 가능한 주 데이터베이스로 전환.

1. #### 보조 리전에서 쓰기 작업 수행:
    * 모든 읽기 및 쓰기 요청은 보조 리전에서 처리됩니다.

1. #### 주 리전 복구:
    * 주 리전이 복구되면, Auto-Failover Group을 사용하여 주 리전으로 다시 장애 조치(failback)를 수행합니다.

    * 주 리전 데이터베이스: 다시 쓰기 가능한 주 데이터베이스로 전환.

    * 보조 리전 데이터베이스: 다시 읽기 전용 복제본으로 설정.

### 구체적인 설정과 동작 방식
1. #### 정상 상태에서의 설정
    * 주 리전(Azure SQL Database): 기본 데이터베이스 (읽기 및 쓰기 가능)

    * 보조 리전(Azure SQL Database): 읽기 전용 복제본

    ```shell
    az sql db replica create --name primaryDB --resource-group myResourceGroup --server primaryServer --partner-server secondaryServer --partner-database secondaryDB --secondary-type Geo --failover-group myFailoverGroup
    ```

1. #### 장애 발생 시의 자동 장애 조치
    * 주 리전 데이터베이스가 장애 상태가 되면, Auto-Failover Group이 보조 리전의 데이터베이스를 쓰기 가능한 상태로 전환합니다.

    ```shell
    az sql failover-group failover --name myFailoverGroup --resource-group myResourceGroup --server secondaryServer
    ```

1. #### 주 리전 복구 후의 장애 조치 복귀
    * 주 리전이 복구된 후, Auto-Failover Group을 사용하여 주 리전으로 다시 장애 조치를 수행합니다. 보조 리전 데이터베이스는 다시 읽기 전용 복제본이 됩니다.

    ```shell
    az sql failover-group failover --name myFailoverGroup --resource-group myResourceGroup --server primaryServer
    ```

### 핵심 요점
* 주 리전 장애 시: 보조 리전의 데이터베이스가 주 데이터베이스로 전환되어 쓰기 작업을 처리합니다.
* 장애 조치 후: 보조 리전에서 모든 읽기 및 쓰기 요청을 처리합니다.
* 주 리전 복구 시: 주 리전으로 다시 장애 조치하여 원래 상태로 복귀합니다.
    
이를 통해 서비스의 가용성을 높이고, 장애 상황에서도 지속적으로 데이터를 처리할 수 있습니다. 장애 발생 시 보조 리전의 데이터베이스가 주 리전 데이터베이스로 라우팅되는 것이 아니라, 보조 리전 데이터베이스 자체가 쓰기 가능한 주 데이터베이스로 전환된다는 점이 중요합니다.


---
---


주 리전에 장애가 발생하기 전까지 보조 리전의 마이크로서비스들은 읽기 작업만 수행합니다. 그러나 주 리전에 장애가 발생하고 보조 리전으로 장애 조치(failover)가 이루어지면, 보조 리전의 마이크로서비스들이 읽기와 쓰기 작업을 모두 수행할 수 있게 됩니다.

### 정상 상태
* 주 리전:
    * 마이크로서비스: 읽기 및 쓰기 작업을 처리.
    * 데이터베이스: 읽기 및 쓰기 가능.

* 보조 리전:
    * 마이크로서비스: 주로 읽기 작업만 수행.
    * 데이터베이스: 읽기 전용 복제본.

### 주 리전 장애 발생 시
1. #### 장애 조치(failover):
    * Auto-Failover Group이 보조 리전으로 장애 조치를 수행.
    * 보조 리전의 데이터베이스가 쓰기 가능 상태로 전환.

1. #### 보조 리전 상태:
    * 마이크로서비스: 읽기 및 쓰기 작업을 모두 수행.
    * 데이터베이스: 읽기 및 쓰기 가능.

### 장애 조치 후의 상황
1. #### 주 리전 장애 발생 전
    * 주 리전:
        * 주문 서비스(Order Service)가 Azure SQL Database에 쓰기 및 읽기 작업 수행.
        * 실시간 배송 추적 서비스(Delivery Tracking Service)가 Azure Cosmos DB에 읽기 및 쓰기 작업 수행.

    * 보조 리전:
        * 주문 서비스가 Azure SQL Database의 읽기 전용 복제본에 대해 읽기 작업만 수행.
        * 실시간 배송 추적 서비스가 Azure Cosmos DB의 읽기 전용 복제본에 대해 읽기 작업만 수행.

1. #### 주 리전 장애 발생 시
    * Auto-Failover Group이 보조 리전의 Azure SQL Database를 쓰기 가능 상태로 전환.

    * 보조 리전의 Azure Cosmos DB가 쓰기 가능 상태로 전환(기본적으로 다중 리전 쓰기 가능).

    * 보조 리전:
        * 주문 서비스가 Azure SQL Database에 대해 읽기 및 쓰기 작업 수행.
        * 실시간 배송 추적 서비스가 Azure Cosmos DB에 대해 읽기 및 쓰기 작업 수행.

### 장애 조치 설정 예시
1. #### Azure SQL Database Auto-Failover Group 설정:

    ```shell
    az sql failover-group create --name myFailoverGroup --resource-group myResourceGroup --server primaryServer --partner-server secondaryServer
    az sql failover-group update --name myFailoverGroup --resource-group myResourceGroup --server primaryServer --add-db primaryDB
    ```

1. #### 주 리전 장애 시 보조 리전으로의 장애 조치:

    ```shell
    az sql failover-group failover --name myFailoverGroup --resource-group myResourceGroup --server secondaryServer
    ```

1. #### 주 리전 복구 후 장애 조치 복귀:

    ```shell
    az sql failover-group failover --name myFailoverGroup --resource-group myResourceGroup --server primaryServer
    ```

### 요약
* 주 리전 정상 상태: 보조 리전의 마이크로서비스는 읽기 작업만 수행.

* 주 리전 장애 시: 보조 리전의 마이크로서비스는 읽기 및 쓰기 작업을 수행.

* 주 리전 복구 후: 주 리전으로 장애 조치(failback)를 수행하여 원래 상태로 복귀.

이를 통해 고가용성을 확보하고 장애 발생 시에도 지속적으로 서비스를 운영할 수 있습니다.


---
---


클라우드에서 다중 리전에 클러스터를 구성하는 경우, 주 리전과 보조 리전 형태로 구성하는 방식은 일반적이지만, 전체적인 요구사항 및 서비스의 성격에 따라 다양한 아키텍처 패턴이 사용될 수 있습니다. 주 리전과 보조 리전 구조는 주로 장애 복구(DR)를 중심으로 한 고가용성(HA) 시나리오에 적합합니다. 다중 리전 아키텍처는 다음과 같은 방식으로 구성될 수 있습니다:

1. ### 주 리전-보조 리전(Primary-Secondary) 아키텍처
    * 주 리전: 모든 쓰기 및 읽기 작업을 처리하는 메인 클러스터.

    * 보조 리전: 주 리전의 데이터 복제본을 유지하고, 주 리전 장애 시 읽기 및 쓰기 작업을 처리.

1. ### 활성-활성(Active-Active) 아키텍처
    * 양쪽 리전: 두 리전 모두에서 읽기 및 쓰기 작업을 동시에 처리.

    * 데이터베이스 복제: 다중 마스터 또는 글로벌 데이터베이스 복제를 사용하여 양쪽 리전의 데이터 일관성을 유지.

    * 트래픽 관리: Azure Traffic Manager, Azure Front Door 등을 사용하여 트래픽을 양쪽 리전에 분산.

1. ### 활성-대기(Active-Passive) 아키텍처
    * 주 리전: 모든 읽기 및 쓰기 작업을 처리.

    * 대기 리전: 평소에는 대기 상태에 있다가 주 리전 장애 시 활성화되어 작업을 처리.

    * 비용 절감: 대기 리전은 리소스 사용을 최소화하여 비용을 절감.

1. ### 리전별 특화 서비스(Region-Specific Services) 아키텍처
    * 리전별 서비스 배포: 각 리전에 특정 서비스만 배포하고, 글로벌 트래픽을 특정 리전으로 라우팅.

    * 데이터베이스 분할: 각 리전에서 독립적인 데이터베이스를 사용하거나, 글로벌 데이터베이스를 사용해 일부 데이터만 복제.

## 주 리전-보조 리전 아키텍처의 예시 설계
1. ### 애플리케이션 배포
    * Azure Kubernetes Service (AKS):
        * 주 리전과 보조 리전에 독립적인 AKS 클러스터를 배포합니다.

1. ### 데이터베이스 설정
    * Azure SQL Database:
        * 주 리전: 읽기 및 쓰기 가능.

        * 보조 리전: 읽기 전용 복제본을 유지.

        * Auto-Failover Group: 장애 발생 시 자동으로 보조 리전으로 장애 조치.

    * Azure Cosmos DB:
        * 다중 지역 배포: 기본적으로 여러 리전에서 읽기 및 쓰기 가능.

        * 자동 장애 조치: 리전 장애 시 자동으로 다른 리전에서 읽기 및 쓰기 처리.

1. ### 트래픽 관리
    * Azure Front Door:
        * 글로벌 트래픽을 주 리전과 보조 리전에 분산하고, 장애 시 자동으로 트래픽을 전환.

1. ### 모니터링 및 복구
    * Azure Monitor 및 Application Insights:
        * 각 리전의 서비스 상태를 모니터링하고, 장애 발생 시 경고 및 자동 복구 절차를 트리거.

## 요약
* 주 리전-보조 리전 아키텍처:
    * 주 리전에서 주로 모든 작업을 처리하고, 보조 리전은 주로 읽기 전용 복제본을 유지하다가 주 리전 장애 시 활성화됩니다.

    * Active-Active 아키텍처: 두 리전 모두에서 동시에 작업을 처리하는 방식으로, 더 높은 가용성과 성능을 제공합니다.

    * Active-Passive 아키텍처: 보조 리전이 대기 상태에 있다가 장애 시 활성화되어 비용 효율성을 높입니다.

각 아키텍처 패턴은 특정 요구사항과 운영 환경에 따라 선택되며, 주 리전-보조 리전 형태는 장애 복구와 고가용성을 확보하는 데 효과적입니다.


---
---


주 리전-보조 리전 아키텍처와 Active-Passive 아키텍처는 유사한 개념으로 보일 수 있지만, 약간의 차이가 있습니다. 두 아키텍처 모두 고가용성(HA)과 장애 복구(DR)를 위한 구조를 제공하지만, 사용 사례와 운영 방식에 약간의 차이가 있습니다.

## 주 리전-보조 리전 아키텍처
**주 리전-보조 리전 아키텍처(Primary-Secondary Architecture)**는 주 리전과 보조 리전 간의 명확한 역할 분담이 있습니다.

* ### 주 리전(Primary Region):
    * 모든 쓰기 및 읽기 작업을 처리합니다.

    * 주 애플리케이션과 데이터베이스가 주 리전에 위치합니다.

* ### 보조 리전(Secondary Region):
    * 주 리전의 데이터를 읽기 전용으로 복제합니다.

    * 주 리전이 정상일 때는 읽기 작업만 수행하거나 대기 상태를 유지합니다.

    * 주 리전에 장애가 발생하면 보조 리전으로 장애 조치(Failover)하여 읽기 및 쓰기 작업을 처리합니다.

* ### 장애 조치 후:
    * 보조 리전이 새로운 주 리전이 되어 모든 쓰기 및 읽기 작업을 처리합니다.

    * 원래 주 리전이 복구되면, 다시 주 리전 역할로 복귀할 수 있습니다.

## Active-Passive 아키텍처
Active-Passive 아키텍처는 주 리전-보조 리전 아키텍처와 매우 유사하지만, 주로 대기 상태에 있는 보조 리전의 활용 방법에 약간의 차이가 있습니다.

* ### Active Region:
    * 모든 쓰기 및 읽기 작업을 처리합니다.

    * 주 애플리케이션과 데이터베이스가 활성 리전에 위치합니다.

* ### Passive Region:
    * 평소에는 대기 상태를 유지합니다.

    * 장애 조치가 필요할 때까지 쓰기 작업을 처리하지 않습니다.

    * 주 리전에 장애가 발생하면, 보조 리전이 활성화되어 모든 쓰기 및 읽기 작업을 처리합니다.

* ### 장애 조치 후:
    * Passive Region이 활성화되어 모든 쓰기 및 읽기 작업을 처리합니다.

    * 원래 Active Region이 복구되면, 다시 주 리전 역할로 복귀할 수 있습니다.

## 차이점 요약
1. ### 주 리전-보조 리전 아키텍처:
    * 보조 리전이 읽기 전용 복제본을 유지합니다.

    * 주 리전 장애 시 보조 리전으로 장애 조치하여 보조 리전이 주 리전 역할을 수행합니다.

    * 보조 리전은 주로 읽기 작업만 처리하거나 대기 상태를 유지하지만, 주 리전 장애 시 쓰기 작업도 처리합니다.

1. ### Active-Passive 아키텍처:
    * Passive Region은 평소에 완전히 대기 상태를 유지합니다.

    * 장애 시 Passive Region이 활성화되어 모든 작업을 처리합니다.

    * Active Region이 복구되면 원래 역할로 복귀합니다.

## 적용 사례
* ### 주 리전-보조 리전 아키텍처:
    * 데이터를 실시간으로 복제하고, 장애 시 신속한 전환이 필요한 경우.

    * 주 리전과 보조 리전 간의 데이터 일관성을 유지하는 것이 중요한 경우.

* ### Active-Passive 아키텍처:
    * 비용 절감이 중요하며, 대기 리전에서 리소스를 최소한으로 유지하는 경우.

    * 장애 복구를 위한 복잡한 설정을 피하고, 단순한 전환이 필요한 경우.

## 결론
주 리전-보조 리전 아키텍처와 Active-Passive 아키텍처는 모두 고가용성과 장애 복구를 제공하지만, 보조 리전 또는 대기 리전의 활용 방식과 데이터 복제 및 장애 조치 방식에 따라 차이가 있습니다. 요구 사항과 운영 환경에 맞는 아키텍처를 선택하는 것이 중요합니다.