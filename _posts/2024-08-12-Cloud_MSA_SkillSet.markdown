---
layout: post
title:  "Cloud MSA를 위한 기술스택"
date:   2024-08-12 23:00:00 +0900
categories: architecture
---

마이크로서비스 아키텍처를 설계할 때, 각 서비스가 독립적으로 배포되고, 확장되며, 관리될 수 있도록 설계하는 것이 중요합니다. 클라우드 기반 애플리케이션에서는 특히 확장성, 가용성, 유연성을 고려해야 합니다. 주문/결제 서비스와 상품리뷰 서비스에 각각 RDB와 NoSQL을 사용하는 요구 사항을 반영한 기술 스택을 아래와 같이 설계할 수 있습니다.

### 마이크로서비스 아키텍처 개요
1.  API Gateway: 클라이언트 요청을 적절한 마이크로서비스로 라우팅하며, 인증, 로깅, 모니터링 등의 공통 기능을 제공합니다.
1.  서비스 디스커버리: 각 마이크로서비스의 위치를 관리하여 동적으로 서비스의 위치를 발견하고, 통신할 수 있도록 합니다.
1.  분산 트랜잭션 관리: 마이크로서비스 간의 트랜잭션을 관리합니다.
1.  데이터베이스: 서비스의 특성에 맞는 데이터베이스를 선택하여 사용합니다.
1.  모니터링 및 로깅: 시스템의 상태를 모니터링하고, 로그를 중앙에서 관리합니다.
1.  CI/CD: 지속적인 통합 및 배포 파이프라인을 구축합니다.

### 주문/결제 서비스 (Order/Payment Service)
* 언어 및 프레임워크: Java + Spring Boot
* 데이터베이스: Amazon RDS (MySQL 또는 PostgreSQL)
* API Gateway: AWS API Gateway
* 서비스 디스커버리: Netflix Eureka
* 분산 트랜잭션: Saga Pattern (Spring Cloud Sleuth, Zipkin)
* 메시지 브로커: RabbitMQ 또는 Apache Kafka
* 모니터링 및 로깅: Prometheus + Grafana, ELK Stack (Elasticsearch, Logstash, Kibana)
* CI/CD: Jenkins, GitHub Actions

### 상품리뷰 서비스 (Product Review Service)
* 언어 및 프레임워크: Node.js + Express
* 데이터베이스: Amazon DynamoDB 또는 MongoDB Atlas
* API Gateway: AWS API Gateway
* 서비스 디스커버리: Netflix Eureka
* 분산 트랜잭션: Event Sourcing (CQRS 패턴)
* 메시지 브로커: RabbitMQ 또는 Apache Kafka
* 모니터링 및 로깅: Prometheus + Grafana, ELK Stack
* CI/CD: Jenkins, GitHub Actions

### 공통 인프라
1. #### API Gateway:
* AWS API Gateway: 각 마이크로서비스로의 요청을 라우팅하고, 인증/인가를 처리합니다.
* Kong 또는 NGINX: API Gateway로 사용할 수 있는 오픈 소스 솔루션.

1. #### 서비스 디스커버리:
* Netflix Eureka: 마이크로서비스의 위치를 관리하고, 동적으로 서비스 발견을 처리합니다.
* Consul: 서비스 메쉬 솔루션으로도 활용 가능.

1. #### 분산 트랜잭션 관리:
* Saga Pattern: 각 서비스가 자체적으로 트랜잭션을 관리하고, 실패 시 롤백을 처리.
* Spring Cloud Sleuth + Zipkin: 분산 트랜잭션 추적을 위해 사용.

1. #### 메시지 브로커:
* RabbitMQ: 주문 생성, 결제 처리 등 이벤트 기반 아키텍처를 지원.
* Apache Kafka: 높은 처리량을 요구하는 이벤트 스트리밍에 적합.

1. #### 모니터링 및 로깅:
* Prometheus + Grafana: 시스템 모니터링 및 시각화를 위해 사용.
* ELK Stack: 로그 수집, 분석 및 시각화를 위해 사용.
* AWS CloudWatch: AWS 기반 모니터링 솔루션.

1. #### CI/CD:
* Jenkins: 빌드, 테스트, 배포 파이프라인을 자동화.
* GitHub Actions: GitHub 리포지토리에 통합된 CI/CD 워크플로우.

### 아키텍처 다이어그램
1. API Gateway가 클라이언트 요청을 Order/Payment Service와 Product Review Service로 라우팅.
1. 각 서비스는 Service Discovery를 통해 서로의 위치를 동적으로 확인.
1. Order/Payment Service는 Amazon RDS에 데이터 저장.
1. Product Review Service는 Amazon DynamoDB 또는 MongoDB에 데이터 저장.
1. RabbitMQ 또는 Kafka를 통해 서비스 간 이벤트를 처리.
1. Prometheus와 Grafana로 각 서비스의 메트릭을 모니터링.
1. ELK Stack으로 로그를 중앙에서 관리.
1. Jenkins와 GitHub Actions를 통해 CI/CD 파이프라인을 구성.

### 결론
이 설계를 통해 각각의 마이크로서비스가 독립적으로 개발, 배포, 확장될 수 있으며, RDB와 NoSQL 데이터베이스 요구 사항을 충족할 수 있습니다. 클라우드 기반 인프라를 활용하여 확장성과 가용성을 높이고, 모니터링 및 로깅을 통해 시스템의 안정성을 유지할 수 있습니다.


---
---


클라우드 환경으로의 변화는 소프트웨어 아키텍처와 관련된 여러 측면에서 중요한 변화를 요구합니다. 기존의 소프트웨어 아키텍트들은 다음과 같은 방식으로 변화해야 합니다:

1. 분산 시스템 이해: 클라우드 환경은 분산 시스템으로 구성되어 있습니다. 아키텍트는 분산 컴퓨팅 모델, 마이크로서비스 아키텍처, 컨테이너화 기술 등을 잘 이해하고 있어야 합니다. 이는 시스템의 확장성, 탄력성, 안정성을 보장하는 데 중요합니다.

1. 서비스 지향 아키텍처 (SOA): 클라우드 환경에서는 서비스 지향 아키텍처가 매우 중요합니다. 서비스는 독립적으로 배포, 확장, 관리될 수 있어야 하며, 다양한 클라우드 서비스들과 통합되어야 합니다.

1. 자동화 및 인프라스트럭처 관리: 클라우드에서는 인프라스트럭처 자동화와 관리가 필수적입니다. 아키텍트는 인프라스트럭처를 코드로 관리하는 Infrastructure as Code (IaC)와 관련된 도구와 기술을 이해하고 적용할 수 있어야 합니다.

1. 탄력적인 아키텍처 구성: 클라우드는 탄력적인 환경을 제공합니다. 아키텍트는 자원의 동적 확장과 축소를 관리하고, 자동화된 스케일링 메커니즘을 구현할 수 있어야 합니다.

1. 보안 및 규정 준수: 클라우드 환경에서는 보안이 매우 중요합니다. 아키텍트는 클라우드 제공자가 제공하는 보안 기능을 이해하고, 애플리케이션 및 데이터의 보안을 유지할 수 있는 방법을 고려해야 합니다. 또한, 규정 준수 요구사항을 충족시킬 수 있도록 설계해야 합니다.

1. 서비스 수준 계약 (SLA) 관리: 클라우드 환경에서는 서비스 제공자와의 SLA를 관리하는 것이 중요합니다. 아키텍트는 서비스의 가용성, 성능, 비용 등을 고려하여 적절한 SLA를 설정하고 관리해야 합니다.

1. 모니터링 및 분석: 클라우드에서는 시스템의 모니터링과 분석이 매우 중요합니다. 아키텍트는 실시간 모니터링 도구와 시스템 성능 데이터를 분석하여 문제를 식별하고 예측할 수 있는 능력이 필요합니다.

이와 같이 클라우드 환경으로의 변화는 기존의 소프트웨어 아키텍트들에게 다양한 새로운 기술과 접근 방식을 요구합니다. 클라우드의 장점을 최대한 활용하고, 시스템의 유연성과 확장성을 제공하는 아키텍처를 설계하고 구현하는 데 주력해야 합니다.