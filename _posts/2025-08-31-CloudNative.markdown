---
layout: post
title:  "Cloud Native"
date:   2025-08-31 20:00:00 +0900
categories: Cloud
---

“Cloud-Native”를 buzzword가 아니라 __실무 설계·운영 방법론__ 으로 풀어볼게요.  
(정의 → 핵심 원칙 → 아키텍처 구성 → 필수 패턴 → 데이터/보안 → 안티패턴 → 도입 로드맵 → 간단 예시 → FlutterFlow/Supabase 맥락)

## 1) 한 줄 정의

Cloud-Native는 컨테이너·마이크로서비스·불변 인프라·선언적 API를 바탕으로, __자동화(CI/CD)와 관측성__ 을 통해 탄력적으로 확장/복구되는 시스템을 설계·구축·운영하는 방식입니다.  
목표는 __빠른 배포, 고가용성/복원력, 운영 자동화__ 예요.

## 2) 핵심 원칙(요약)

* __컨테이너화__: 실행 환경을 이미지로 표준화(이식성↑)

* __마이크로서비스__: 단일 책임, 독립 배포(결합↓, 응집↑)

* __선언적 구성__: IaC(Git), 쿠버네티스 리소스(YAML), GitOps

* __불변(Immutable) 인프라__: SSH로 고치는 대신 새 버전 재배포

* __자동화된 전달 파이프라인__: CI/CD, Canary/Blue-Green, Feature Flag

* __관측성(Observability)__: 로그·메트릭·트레이싱(분산 추적)

* __자가치유/탄력성__: 오토스케일, 재시도/서킷브레이커, 헬스체크

* __보안 내재화(DevSecOps)__: SBOM/서명, mTLS, 비밀관리, 정책코드(OPA)

* __SRE 지표 기반 운영__: SLI/SLO, 에러버짓, 무중단/점진 배포

* __비용 가시화(FinOps)__: 요청/Pod/스토리지 단위로 비용 추적·최적화

## 3) 아키텍처 레이어

* __앱 레이어__: 도메인 서비스(REST/gRPC/Event), 이벤트 핸들러

* __플랫폼__: Kubernetes(스케줄러/서비스/Ingress/Job/CronJob)

* __Runtime__: 컨테이너 런타임, 이미지 레지스트리

* __전달 파이프라인__: CI(Coverage/보안스캔) → CD(GitOps/ArgoCD)

* __네트워킹__: Ingress/Service Mesh(Istio/Linkerd), mTLS, 트래픽 분할

* __관측성__: Prometheus, Loki/ELK, OpenTelemetry, Grafana

* __데이터__: StatefulSet+PV/PVC, 오퍼레이터(DB 운영 자동화), 백업/DR

## 4) 필수 설계 패턴

* 12-Factor & Externalized Config(환경변수/시크릿)

* Idempotency/Retry with Backoff/Time-outs(네트워크 안정성)

* Circuit Breaker/Bulkhead(연쇄 장애 차단)

* Canary/Blue-Green/Feature Flag(점진적 위험관리)

* Event-Driven(비동기 확장, Kafka/Queue)

* GitOps(모든 변경=PR, 선언적 동기화, “실제=Git에 적힌 상태”)

## 5) 상태(State)와 데이터

* __Stateless 앱__: 세션·진행상태는 토큰/스토리지/백엔드에 둠 → 수평 확장 쉬움

* __Stateful 워크로드__: DB/큐/검색은 오퍼레이터(예: Postgres/MySQL Operator)로 배포·백업·업그레이드 자동화

* __DR 설계__: RPO/RTO 목표, 증분 백업, 스냅샷, 멀티-AZ/리전, Read Replica

* __일관성 모델__: 최종 일관성 + UX 보정(“처리중”/메모리밸런스)

## 6) 보안(Shift-Left)

* __이미지 보안__: SBOM, 취약점 스캔, 이미지 서명(SLSA/키 체인)

* __비밀 관리__: KMS/Sealed-Secrets/External Secrets

* __네트워크__: mTLS(Service Mesh), 네임스페이스·네트워크폴리시

* __정책 코드__: OPA/Gatekeeper/Kyverno로 배포 정책 강제

* __런타임__: 최소 권한, Seccomp/AppArmor, Falco 등 런타임 감시

## 7) 안티패턴(피해야 할 것)

* Lift-and-Shift 후 VM + 수동 설정 유지(불변/선언적 X)

* 상태ful session(Sticky)·로컬 디스크 의존(스케일/복구 어려움)

* 단일 거대 서비스(변경 파급↑), 릴리즈마다 전체 재배포

* 모든 트래픽을 동기식으로만 처리(스파이크에 취약)

* 모니터링/추적 없이 장애만 나면 수작업 탐사(Mean-time-to-repair↑)

## 8) 도입 로드맵(현실적인 순서)

1. __컨테이너화__: 빌드 표준화, 설정 외부화(12-Factor)

1. __CI__: 테스트/보안스캔/커버리지, 이미지 레지스트리

1. __쿠버네티스 배포__: Deployment/Service/Ingress, 헬스체크

1. __관측성 기본__: 메트릭·로그·트레이스 + 대시보드/알람

1. __CD/GitOps__: PR→자동 배포, 롤백 원클릭, Canary 도입

1. __데이터 운영성__: 오퍼레이터/백업/DR, 성능·쿼터 관리

1. __서비스 메쉬(필요할 때)__: mTLS, 트래픽 분할, 정책 일원화

1. __SRE 운영__: SLI/SLO/에러버짓, 혼돈테스트(Chaos)로 복원력 검증

1. __FinOps__: 오토스케일 튜닝(HPA/KEDA), 리소스/비용 대시보드

## 9) 최소 예시 (Kubernetes 배포 + 오토스케일)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: orders, labels: { app: orders, version: v1 } }
spec:
  replicas: 2
  selector: { matchLabels: { app: orders } }
  template:
    metadata: { labels: { app: orders, version: v1 } }
    spec:
      containers:
        - name: app
          image: ghcr.io/acme/orders:v1.3.7
          ports: [{ containerPort: 8080 }]
          env:                      # 외부화된 설정
            - { name: DB_URL, valueFrom: { secretKeyRef: { name: orders-secrets, key: db } } }
          readinessProbe: { httpGet: { path: /healthz, port: 8080 }, periodSeconds: 5 }
          livenessProbe:  { httpGet: { path: /livez,   port: 8080 }, periodSeconds: 10 }
---
apiVersion: v1
kind: Service
metadata: { name: orders }
spec:
  selector: { app: orders }
  ports: [{ port: 80, targetPort: 8080 }]
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: orders-hpa }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: orders }
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 60 } }
```

* 불변 배포(이미지 태그), 헬스체크, HPA로 오토스케일 → cloud-native 기본기

## 10) FlutterFlow + Supabase에 대입

* __앱__: 완전 Stateless 화면/액션, Edge Functions로 도메인 로직 분리

* __데이터__: RLS로 보안, RPC(원자 동작)로 트랜잭션 캡슐화

* __전달__: Functions를 GitHub Actions로 빌드/배포(버전 태깅), Canary 키(Feature Flag)로 점진 롤아웃

* __관측성__: Edge Logs + DB 모니터링(쿼리 시간/잠금), 에러추적(Sentry)

* __캐시/확장__: Redis 등으로 읽기 가속, KEDA로 큐 길이에 따른 스케일

## 마무리

Cloud-Native의 본질은 “선언적·자동화·관측 가능한 운영으로 작게 빠르게 바꾸면서도 안정성을 확보”하는 겁니다.

---

MSA 관점에서 Cloud-Native를 “실무 설계·운영 방식”으로 풀어볼게요. 핵심은 작게 나눈 서비스를 컨테이너로 표준화하고, 선언적 인프라+자동화(CI/CD/GitOps) 위에서 탄력적으로 확장·복구되게 만드는 것입니다.

## 1) Cloud-Native가 MSA에 주는 것

* __독립 배포__: 서비스마다 이미지 태그로 릴리스, 롤백·카나리 쉽게.

* __수평 확장__: Stateless + 오토스케일(HPA/KEDA)로 트래픽 스파이크 흡수.

* __장애 격리__: 실패가 다른 서비스로 연쇄 전파되지 않도록 Bulkhead/서킷브레이커.

* __운영 표준화__: 선언적 리소스(YAML), GitOps(ArgoCD 등)로 “실제 상태 = Git”.

## 2) MSA × Cloud-Native 핵심 원칙

* __경계/응집__: 도메인 기준으로 서비스 분리(SRP). “하나의 변경 이유”.

* __Stateless by default__: 세션·흐름 상태는 토큰/스토리지/백엔드로 외부화.

* __DB-per-service__: 스키마 공유 금지(데이터 복제 허용). 일관성은 SAGA/CQRS로.

* __계약 우선__: REST/gRPC 스키마 고정, 버저닝, Backward-compatible 변화.

* __비동기 우선__: 큐/Kafka로 decouple. 동기 호출엔 타임아웃·재시도·서킷브레이커.

* __관측성 내장__: 로그·메트릭·트레이스(OpenTelemetry) + Correlation ID.

* __보안 내재화__: mTLS(Service Mesh), Secret 관리, 최소권한/RBAC, 정책코드(OPA).

* __불변 인프라__: 수동 SSH 패치 X → 새 이미지로 재배포.

## 3) 준참조 아키텍처(논리)
```scss
[Client] → [API Gateway] → [Service A]──┐
                             │          ├─[Redis 캐시]
                             ├─(gRPC/HTTP)→[Service B]→[DB_B]
                             └─(이벤트)→[Kafka/Queue]→[Workers]
      ↑   observability: OTEL(Trace) + Prometheus(Metrics) + Loki/ELK(Logs)
      └─ security: mTLS(Service Mesh), OPA 정책, Vault/Secrets
```

## 4) 데이터 & 일관성 패턴

* __CQRS__: 쓰기 모델(정합성)과 읽기 모델(성능)을 분리, 읽기엔 캐시/머티리얼라이즈드 뷰.

* __SAGA(보상 트랜잭션)__: 분산 트랜잭션 대신 “앞으로 진행 → 실패 시 역순 보상”.

* __Outbox + CDC__: 같은 트랜잭션에서 이벤트를 Outbox에 기록→워커가 브로커로 전송(유실 방지).

* __Idempotency__: 생성/결제/예약 등엔 키로 멱등 보장(중복 호출 안전).

## 5) 안정성 패턴(실무 필수)

* __Timeout/Retry with backoff/Jitter__: 네트워크 기본기.

* __Circuit Breaker__: 상대 서비스/DB 장애시 빠른 페일·격리.

* __Bulkhead/Pool 분리__: 스레드/커넥션 풀 분리로 한 기능이 전체를 잠식하지 않게.

* Rate limit(게이트웨이·엣지), 캐시(Cache-aside, SWR, 분산락)로 스파이크 흡수.

## 6) 배포 & 운영

* __CI__: 테스트/커버리지/취약점 스캔 → 이미지 레지스트리.

* __CD/GitOps__: PR 머지 → ArgoCD/Flux가 클러스터 동기화. Canary/Blue-Green, Feature Flag.

* __오토스케일__: HPA(CPU/메모리) + KEDA(큐 길이·커스텀 메트릭).

* __SRE 지표__: SLI/SLO(가용성·지연·오류율·신뢰 구간), 에러버짓으로 출시 속도 관리.

* __FinOps__: 요청/제한, Pod/스토리지 단위 비용 가시화, Right-sizing/스팟/스케일-투-제로.

## 7) 보안

* 서비스 간 mTLS(Service Mesh) + 제로트러스트.

* __토큰 기반 권한(OAuth2/OIDC/JWT)__, 최소 권한의 서비스 계정.

* __비밀__: KMS/Vault/Sealed Secrets, 회전 자동화.

* __정책__: OPA/Kyverno로 “이미지 서명, 런타임 보안, 네트워크 정책” 강제.

## 8) 안티패턴(경계주의)

* 공용 DB 스키마(강결합, 스키마 충돌).

* 모든 호출을 동기로 직렬화(꼬리 지연 ↑, 장애 전파 ↑).

* 무버전 API/스키마(클라이언트 파손).

* 운영 미관측(분산 추적·대시보드 부재).

* “마이크로서비스 수만 늘림”(업무 경계 없이 쪼갬).

## 9) 최소 배포 예시(YAML 스케치)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: products, labels: { app: products } }
spec:
  replicas: 2
  selector: { matchLabels: { app: products } }
  template:
    metadata:
      labels: { app: products }
      annotations: { "prometheus.io/scrape": "true", "sidecar.istio.io/inject": "true" }
    spec:
      containers:
        - name: app
          image: ghcr.io/acme/products:v1.8.2
          envFrom: [{ secretRef: { name: products-secrets } }]
          ports: [{ containerPort: 8080 }]
          resources: { requests: { cpu: "200m", memory: "256Mi" }, limits: { cpu: "1", memory: "512Mi" } }
          readinessProbe: { httpGet: { path: /healthz, port: 8080 }, periodSeconds: 5 }
          livenessProbe:  { httpGet: { path: /livez,   port: 8080 }, periodSeconds: 10 }
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: products-hpa }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: products }
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 60 } }
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: { name: products-pdb }
spec: { minAvailable: 1, selector: { matchLabels: { app: products } } }
```

* 헬스체크/자원요청/HPA/PDB: 기본기만 갖춰도 가용성·탄력성이 급상승합니다.

## 10) FlutterFlow + Supabase 맵핑(당신의 맥락)

* __서비스 경계__: “상품/주문/결제/계정” 등 Edge Function 단위로 분리(각각 독립 배포).

* __DB 경계__: 실제 단일 Postgres라도 스키마 분리(예: products.*, orders.*) + RLS로 격리.

* __비동기__: 트리거/Outbox → Supabase Realtime/큐로 이벤트 발행, 장기 작업은 워커(스케줄러)로.

* __일관성__: 장바구니→주문→결제에 SAGA 보상(예약해제/VOID/환불) 도입.

* __관측성__: Edge Logs + SQL 통계 + Sentry(OpenTelemetry SDK)로 추적.

### 도입 로드맵(빠른 실행 6단계)

1. 도메인 경계 재정의(서비스 캔버스) + API 계약/스키마 버전 결정

1. 컨테이너화 & 12-Factor(설정 외부화, 상태 제거)

1. CI/CD + GitOps(PR→이미지→배포 자동) + 기본 모니터링

1. HPA/KEDA & 캐시(핫경로 가속, 큐 길이 기반 스케일)

1. 비동기화(Outbox/SAGA, CDC) + 서킷브레이커/타임아웃

1. 보안·정책·FinOps(mTLS, OPA, 비용대시보드) 정착

---

## 1) Cloud-Native 도입에 잘 맞는 비즈니스 모델

“빠르게 자주 바꾸고, 트래픽이 요동치며, 글로벌로 확장해야 하는” 모델일수록 이득이 큽니다.

* __SaaS(멀티테넌트)__: 고객별 테넌트 격리·자동 확장·버전 롤아웃/롤백이 잦음
    - (예: B2B 업무툴, 분석 플랫폼)

* __이커머스/마켓플레이스__: 시즌/프로모션 스파이크, 지역 확장, 기능 실험(Canary/Feature Flag) 빈번

* __핀테크/결제/구독__: 규제 준수 + 고가용성 + 이벤트 기반 처리(비동기) 필요

* __미디어/게임/라이브 서비스__: 순간 동시접속·저지연·글로벌 엣지 서빙

* __IoT/스트리밍/실시간 데이터__: 이벤트 드리븐·수평 확장 친화

* __온디맨드/물류(배달·모빌리티)__: 수요 변동 큼, 지역/노선/재고 연산 분산 필요

* __API 비즈니스/플랫폼__: 외부 개발자 대상, 버저닝·SLO 관리·관측성이 핵심

반대로 변경이 드물고 트래픽이 작은 단일 업무 시스템, 또는 **강한 데이터 종속(대형 모놀리식 DB, 메인프레임)**만 있는 곳은 초기 ROI가 낮을 수 있어요(점진 도입 권장).

## 2) Cloud-Native 도입 시 고려 사항(체크리스트)
### A. 제품/비즈니스 관점

* __서빙 패턴__: 트래픽 변동·글로벌 확장·신규 기능 실험이 잦은가?

* __SLO/SLA__: 가용성·지연 목표가 명확한가(예: 99.9%, p95=200ms)?

* __릴리스 주기__: 주 1회 이상 배포 필요? 카나리/롤백 요구?

### B. 조직/역량

* __DevOps/SRE 문화__: “빌드→배포→운영” 자동화와 온콜 체계를 감당할 팀?

* __플랫폼 엔지니어링__: 쿠버네티스·CI/CD·관측성 스택을 운영할 코어팀 존재?

* __서비스 경계 정의__: 도메인 기준 MSA 설계 역량(응집↑, 결합↓)?

### C. 아키텍처/데이터

* __Stateless 우선__: 세션/흐름 상태 외부화(토큰/스토리지/백엔드)

* DB-per-service와 데이터 일관성 패턴(SAGA, CQRS, Outbox/CDC) 준비

* __상태ful 워크로드 전략__: DB/큐/검색은 오퍼레이터·백업/DR·복구 연습

### D. 보안/규제

* __제로트러스트__: mTLS/네트워크 정책, 이미지 서명, SBOM/취약점 스캔

* __비밀/키 관리__: KMS/Vault/Sealed-Secrets, 키 회전 자동화

* __컴플라이언스__: 데이터 거주/PCI/PII/감사로그, 변경 이력(GitOps) 증적

### E. 운영/관측성

* __표준 관측성__: 로그·메트릭·트레이스(OpenTelemetry), 대시보드·알람

* __탄력성 패턴__: 타임아웃/재시도·서킷브레이커·Bulkhead·HPA/KEDA

* __릴리스 전략__: Blue-Green/Canary/Feature Flag·원클릭 롤백

### F. 비용/FinOps

* __Right-sizing__: 요청/제한 설정, 오토스케일, 스팟/스케일-투-제로

* __비용 가시화__: 네임스페이스·팀·서비스 단위 쇼백/차지백

* __네트워크/스토리지 비용__: egress·영구 스토리지·레지스트리 트래픽 고려

### G. 마이그레이션 전략(레거시→CN)

* __목표 선정__: 스파이크 노출·변경 빈도 높은 도메인부터(ROI 우선)

* __Strangler 패턴__: 경계부터 분리, 점진적 라우팅 전환

* __6R 프레임__: Rehost→Replatform→Refactor 순으로 현실적 단계화

* __GitOps/IaC 선행__: “선언적 상태 = 실제 상태”를 먼저 확립

## 빠른 실행 로드맵(6단계)

1. 도메인 경계/서비스 캔버스 정리 + API 계약/버전 정책

1. 컨테이너화 & 12-Factor(설정 외부화, 불변 이미지)

1. CI/CD + GitOps(PR→이미지→카나리/롤백 자동화)

1. 관측성 기본(OTel/Prometheus/로그/트레이스) + SLO 수립

1. 비동기화(큐/Kafka, Outbox/CDC, SAGA) + 안정성 패턴

1. FinOps & 보안 내재화(mTLS·OPA·SBOM, 비용 대시보드)

## 도입 적합성 빠른 판단(미니 매트릭스)

* __적합__: 기능 릴리스 주 1회+, 피크 트래픽 ≥ 평시 5배, 글로벌/멀티테넌트, 가용성 99.9%+, 팀에 DevOps 경험 有

* __부분적__: 내부 백오피스·변경 낮음·단일 리전 → 컨테이너화/CI부터 점진 도입

* __보류__: 데이터 중력 강한 모놀리식 + 팀 역량/예산 부족 → 리플랫폼·관측성부터 준비

---

**당신의 현재 맥락(FlutterFlow + Supabase 기반 쇼핑앱, 다국어·동유럽 타깃)**을 전제로

### 1. Cloud-Native 도입 적합성 진단표(점수화)

### 2. 12주 전환 계획(마일스톤 · KPI · 비용 관점)

을 한 번에 드립니다.

## 1) Cloud-Native 도입 적합성 진단표 (0~5점, 총 50점)

|	항목	|	의미(고득점일수록 도입가치↑)	|	베이스라인 점수	|	근거/메모	|
|	---	|	---	|	---	|	---	|
|	기능 출시 속도	|	배포 빈도·실험 필요성	|	4	|	쇼핑앱은 프로모션/번역/카탈로그 변경이 잦음	|
|	트래픽 변동성	|	시즌/딜/이벤트 스파이크	|	4	|	세일·광고 시 피크 발생 가능	|
|	글로벌/지연 요구	|	지역 분산/다국어·로케일	|	4	|	동유럽 타깃 + 한국 운영	|
|	가용성/SLA	|	99.9%+ 등 목표	|	3	|	초기엔 99.5~99.9% 현실적	|
|	이벤트 지향 적합성	|	주문/결제/알림의 비동기화	|	4	|	주문→결제→재고 전형적 이벤트 플로우	|
|	데이터 제약 낮음	|	레거시 의존↓/현대 DB	|	4	|	Supabase(Postgres), 스키마 변경 탄력	|
|	DevOps/SRE 숙련	|	자동화·온콜 역량	|	2	|	(가정) 소규모 팀, 아직 성장 중	|
|	CI/CD 성숙	|	테스트/배포 자동화	|	2	|	Edge Functions/DB 마이그 마이그레이션 자동화 필요	|
|	관측성 성숙	|	로그·메트릭·트레이스	|	2	|	기본 로그 외 분산추적 미구축 가정	|
|	컴플라이언스 민감도	|	결제/개인정보 처리	|	3	|	결제 연동·PII 보관 최소화 지향	|

    총점 = 4+4+4+3+4+4+2+2+2+3 = 32 / 50 (64%) → “적합(중상)”
    → 도입시 이득이 명확, 조직/자동화/관측성을 올리면 “고적합”으로 상승 여지 큼.
    (셀프 점검에 맞춰 각 점수만 조정하시면 총점 재해석이 가능합니다.)

## 2) 12주 전환 계획 (마일스톤 · KPI · 비용 관점)
### 전체 원칙

* 작게·자주: 위험을 쪼개고 Canary/Feature-Flag로 배포

* __Stateless 우선__: 세션·흐름은 토큰/스토리지, 트랜잭션은 **RPC(원자)**로

* __비동기__: Outbox/이벤트로 decouple, 결제·재고는 SAGA 보상

* __관측성/보안 내재화__: 처음부터 OTel/로그/알람 + 시크릿 관리

### W1–W2: 경계·SLO·관측성 “뼈대” 세우기

* 마일스톤

    * __서비스 경계 초안__: catalog / cart / order / payment

    * __SLO 정의__: 가용성(예: 99.9%), p95 지연(예: 300ms), 오류율

    * __관측성 베이스라인__: Edge Functions/클라이언트에 에러 추적(Sentry 등), API Latency/Rate/Error 대시보드

    * Supabase DB 마이그레이션 관리(CLI) 시작: 모든 스키마 변경 PR 기반

* KPI

    * 대시보드 1개(요청수/지연/오류) 완성

    * 마이그 PR→마이그 자동 적용 파이프라인 1개

* 비용 관점

    * __관측성/로그__: 시작은 프리 · 소규모 요금제(월 수십달러 수준)

### W3–W4: CI/CD·릴리스 전략

* 마일스톤

    * Edge Functions/프론트 CI(테스트·린트·번들) → CD(프리뷰/프로덕션)

    * Feature Flag/환경별 시크릿 관리(프로덕션/스테이징 분리)

    * API 계약(스키마) 문서화와 버저닝 정책(v1 유지 · 필드 추가는 호환)

* KPI

    * 배포 소요 15분↓, 롤백 5분↓

    * 메인 브랜치 주 2회↑ 배포

* 비용 관점

    * CI/CD 실행 분당 과금(소액), 시크릿 저장소 무료/저가

### W5–W6: 도메인 분리 + RPC/캐싱 기초

* 마일스톤

    * Supabase RPC로 원자 동작 캡슐화: cart_increment, order_place

    * Idempotency-Key 도입(주문·결제 API)

    * 읽기 경로 캐싱(예: Redis/서드파티) 도입: 카탈로그/번역 키

    * API 게이트웨이 규칙: Rate Limit/HTTP 캐시 헤더

* KPI

    * 주문 API 재시도 시 중복 생성 0건

    * 카탈로그 응답 p95 30%↓, 캐시 Hit 60%↑

* 비용 관점

    * 캐시(작은 플랜) 월 10–50달러 규모부터

### W7–W8: 비동기화(Outbox/이벤트) + SAGA(보상)

* 마일스톤

    * Outbox 테이블 + 워커(크론/Edge)로 TransactionRecorded 발행

    * order → payment(authorize) → inventory(reserve) → 실패 시 VOID/Release 보상 구현

    * 큐/이벤트 소스(예: Supabase Realtime or 외부 큐) 연결

* KPI

    * 큐 적체(대기) p95 < 2s

    * 보상 트랜잭션 실패율 < 0.1% (재시도로 자동 회복)

* 비용 관점

    * 이벤트/큐 소형 플랜 월 수~수십달러

### W9–W10: 보안·정책·테스트 강화

* 마일스톤

    * 최소권한 RLS 재점검, 시크릿 회전/권한 분리(서비스 롤)

    * 계약 테스트(클라이언트↔API), E2E 핵심 플로우(로그인→주문→결제)

    * 혼돈 테스트(간단): 결제/재고 지연 시 SAGA 회복 검증

* KPI

    * 테스트 커버리지(핵심 유스케이스) 80%↑

    * E2E 성공률 99%↑

* 비용 관점

    * 테스트 도구/러너 추가비용 경미

### W11: 성능·용량·Canary

* 마일스톤

    * 부하 테스트(피크 x1.5) 통과, Canary 10% → 100% 전환 플레이북

    * 캐시/DB 인덱스/쿼리 튜닝

* KPI

    * p95 응답 300ms↓(카탈로그), 주문 실패율 < 0.5%

    * 캐시 Hit 70%↑

* 비용 관점

    * 테스트 중 임시 스케일 비용 상승(일시적)

### W12: 프로덕션 롤아웃 & 운영체계

* 마일스톤

    * 운영 Runbook/온콜 · 알람 임계치 설정

    * SLO 준수 리포트 v1 · 사후비용(FinOps) 리뷰

* KPI

    * 가용성 SLO(예: 99.9%) 충족

    * 경보 오탐 < 10%, MTTR 30분↓

* 비용 관점

    * 월간 추정(소규모 기준, 매우 러프)

        * Supabase/DB: 트래픽·스토리지에 따라 수십~수백달러

        * 캐시/이벤트/모니터링: 수십달러대 시작

        * 전체(스테이지+프로덕): 수백~천 달러 초반에서 시작 (트래픽·데이터 크기에 민감)

## 핵심 산출물(이번 12주 내)

* 아키텍처 결정 기록(ADR): 경계, 비동기, 캐시, 보상 정책

* SLO/SLA 문서 + 대시보드(요청·지연·오류·큐 적체·캐시 히트)

* CI/CD 파이프라인 + 릴리스 전략(Feature Flag/Canary)

* 보상 트랜잭션 표(Forward↔Compensation 매핑)

* Runbook/온콜/알람 + 사후비용(코스트) 리포트

## 위험·대응 요약

* 스키마 변경 리스크 → 마이그 PR & 롤백 스크립트, 백업 검증

* 이벤트 유실/중복 → Outbox+멱등키, 재시도/데드레터 큐

* 동시만료/캐시 폭주 → TTL+Jitter, 분산락(single-flight), SWR

* 배포 실패 → Canary/원클릭 롤백, 헬스체크·서킷브레이커