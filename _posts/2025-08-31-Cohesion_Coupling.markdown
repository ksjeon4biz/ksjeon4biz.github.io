---
layout: post
title:  "결합도와 응집도"
date:   2025-08-31 11:45:00 +0900
categories: architecture
---


좋아요! 먼저 용어부터 정리할게요.

### 응집도(Cohesion): 
모듈(클래스/함수/패키지) 내부 요소들이 하나의 목적을 위해 얼마나 잘 뭉쳐 있는가.

### 결합도(Coupling): 
서로 다른 모듈들이 얼마나 강하게 의존하는가. (낮을수록 좋음)

## 응집도(Cohesion): 높을수록 좋다
모듈이 “한 가지 책임”에 집중할수록 응집도가 높습니다.
* 기능적 응집(최상): 하나의 명확한 기능만 수행
* 순차적/통신적 응집: 입력·출력 흐름이 서로 강하게 연결
* 절차적/시간적/논리적 응집
* 우연적 응집(최하): 관련 없는 일들을 한데 모아둔 상태

### 신호
* 함수/클래스 이름이 구체적이고 짧다.
* “그리고/또한”이 반복되는 설명이 없다.
* 변경 사유가 거의 한 가지다(SRP).


## 결합도(Coupling): 낮을수록 좋다
모듈 간 연결을 느슨하게 만들수록 변경에 강합니다.
* 내용 결합(최악): 다른 모듈 내부에 직접 접근/변경
* 공통/외부 결합: 전역변수, 공유 자원에 의존
* 제어 결합: 플래그/분기 신호로 상대 모듈의 흐름을 제어
* 스탬프(구조체) 결합: 필요 이상으로 큰 DTO 전달
* 데이터 결합(이상적): 딱 필요한 데이터만 주고받음

### 낮추는 요령
* 플래그 인자(booleans) 지양 → 전략/다형성으로 분기 제거
* 거대한 DTO 대신 명시적 파라미터 또는 작은 값 객체
* 구현 대신 인터페이스/포트 의존(DIP), DI 적용
* 외부 시스템은 어댑터로 격리(헥사고널/클린 아키텍처)

## 응집도 ↑, 결합도 ↓ 를 동시에 달성하는 방법
* 경계(boundary) 나누기: Orders, Payments, Inventory 등 도메인별 패키지 분리
* 단일 책임(SRP): 서비스가 여러 책임을 지면 응집도↓, 곳곳과 연결돼 결합도↑
* 의존 역전(DIP): 핵심 도메인 → 추상 포트, 외부 인프라 → 어댑터 구현
* 유스케이스 단위 테스트: 포트에 대한 테스트로 변경 비용 축소

미니 예시 (개선 전 → 후)

```python
# 개선 전: 응집도↓, 결합도↑ (한 곳에서 결제/DB/메일까지)
def process_order(order_id, use_test_gateway=False):
    order = db.find(order_id)
    if use_test_gateway:
        ok = TestPG().pay(order.total)
    else:
        ok = RealPG(api_key).pay(order.total)
    if ok:
        db.save(order.mark_paid())
        send_email(order.user, "paid")
```

```python
# 개선 후: 유스케이스에 집중(응집도↑), 외부 의존은 포트로 격리(결합도↓)
class PaymentPort:
    def pay(self, amount: int) -> bool: ...

class ProcessOrder:
    def __init__(self, repo, payment: PaymentPort, mailer):
        self.repo = repo; self.payment = payment; self.mailer = mailer

    def execute(self, order_id):
        order = self.repo.find(order_id)
        if not self.payment.pay(order.total):
            return False
        self.repo.save(order.mark_paid())
        self.mailer.send(order.user, "paid")
        return True
```

* 유스케이스 ProcessOrder는 “주문 처리”에만 집중 → 응집도↑
* 결제/메일/DB는 추상화 뒤로 숨김 → 실제 구현 교체 쉬움 → 결합도↓

## 실무 체크리스트 (짧게)
* 하나의 클래스/함수에 “변경 이유가 하나인지” 확인(SRP)
* 플래그 인자/거대 DTO 줄이기 → 전략/작은 값 객체로 치환
* 외부 API/DB/메시징은 포트-어댑터로 경계화
* 테스트는 유스케이스 단위로, 인프라는 테스트 대역으로 교체
* 주기적으로 의존 방향(도메인 → 추상, 인프라 → 구현) 점검

원하시면 지금 진행 중인 FlutterFlow + Supabase 구조에 맞춰, 화면-서버-DB 경계에서 응집/결합 개선안을 구체적으로 짚어드릴게요.

---

좋아요—“응집도↑, 결합도↓”를 실무에서 바로 점검·개선할 수 있게 체크리스트+처방전 형태로 정리해 드릴게요. (FlutterFlow + Supabase 맥락도 같이 반영)

## 1. 아키텍처 경계 & 의존 방향
### 체크
* 도메인(use-case) → “추상(포트)”에 의존, 인프라(DB/API/UI) → “구현(어댑터)”에 의존하나요? (DIP)
* 패키지 사이 순환 의존이 없나요?
* 안정적인 모듈(많이 참조됨)이 불안정한 모듈에 의존하지 않나요? (Stable Dependencies Principle)

### 신호
도메인 서비스가 supabase, http, flutterflow action 등을 직접 호출
import 그래프에 사이클 존재

### 처방
“포트-어댑터”로 갈라서, 도메인은 다음과 같은 인터페이스만 봅니다:

```python
class CartRepo: 
    def find(self, user_id): ...
    def save(self, cart): ...
class PaymentPort:
    def pay(self, amount): ...
```

Supabase 호출/FlutterFlow 액션은 어댑터에서 구현:
```python
class SupabaseCartRepo(CartRepo):
    def find(self, user_id):  # RPC 호출 or select
    def save(self, cart):     # RPC로 원자 처리
```

## 2. 모듈 응집도 (Cohesion)
### 체크
* 클래스/함수의 “변경 이유”가 한 가지인가요? (SRP)
* 이름에 “그리고/또한/겸해서” 뉘앙스가 들어가나요?
* 함수가 30줄+, 파라미터 5개+, Boolean 플래그가 자주 등장하나요?

### 신호
* OrderService가 결제/영수증/메일/SMS까지 다 처리
* “유틸” 모듈에 잡다한 기능을 계속 추가

### 처방
* 유스케이스 단위로 서비스 분할 (예: PlaceOrder, CancelOrder)
* “플래그 인자”가 나오면 전략/다형성으로 갈라내기
* 큰 함수는 Extract Function → 이름이 기능을 말하도록

## 3. 결합도 (Coupling)
### 체크
* 외부 시스템/라이브러리에 직접 깊게 의존하지 않나요?
* 거대한 DTO(필드 20+)를 여기저기 전달하나요?
* 전역 상태/싱글톤에 의존하나요?

### 신호
* 함수 인자로 config, db, http, logger, featureFlag, ...를 잔뜩 전달
* 테스트에서 외부 IO를 매번 때려야만 함

### 처방
* 필요한 데이터만 전달(데이터 결합)
* 외부 의존은 포트 인터페이스 뒤로 숨기고, 테스트에선 대역 사용
* 싱글톤/전역 대신 주입(의존성 주입)

## 4. 데이터/백엔드 (Supabase 중심)
### 체크
* 테이블 직접 노출 대신 의미 있는 RPC(원자 동작)로 캡슐화했나요?
* RLS 정책이 유스케이스/역할과 정렬되어 있나요?
* “클라이언트에서 다단계 갱신” 대신 DB에서 트랜잭션으로 처리하나요?

### 신호
* 장바구니 수량 증가가: select → 계산 → update(경쟁 조건 위험)
* RLS 미정비로 anon 키에서 쓰기 가능한 경로 존재

### 처방
RPC 예시(원자 증가):

```sql
create or replace function cart_increment(_user uuid, _pid text, _delta int)
returns void language plpgsql security definer as $$
begin
  insert into cart(user_id, pid, qty) values (_user, _pid, greatest(_delta,1))
  on conflict(user_id, pid) do update
    set qty = cart.qty + excluded.qty;
end$$;
```

RLS: auth.uid() = user_id AND role = 'user' 같은 명시적 가드
목록/상세 조회는 필요한 컬럼만 반환하는 뷰/함수로 정리

## 5. 프런트(FlutterFlow) 구조

### 체크
* 페이지가 “한 가지 사용자 목표”에 집중하나요?
* 비즈니스 로직이 위젯 트리/액션 체인에 퍼져있지 않나요?
* App State가 거대하고, side-effect가 여기저기서 일어나지 않나요?

### 신호
* OnPageLoad에서 API 호출, 파싱, 검증, 캐시, 라우팅을 한 번에 수행
* 동일 로직이 여러 위젯 액션에 복붙

### 처방
* Custom Action/Function으로 비즈니스 로직을 모으고, 위젯은 입출력만
* App State는 읽기 전용 뷰 모델 느낌으로 슬림하게 유지
* 네트워크/DB 호출은 한 계층(예: Repository)에서만

## 6. 테스트 전략 (품질-비용 균형)

### 체크
* 유스케이스(도메인) 단위의 빠른 테스트가 핵심인가요?
* 어댑터(Supabase/HTTP)에는 계약 테스트가 있나요?
* E2E는 “핵심 시나리오”에만 얇게?

### 피라미드
* 단위/유스케이스 테스트: 포트 대역으로 빠르게 다수
* 통합/계약 테스트: 실제 Supabase 스키마/RPC 계약 확인
* E2E: 로그인→상품담기→결제 핵심 플로우 소수

### 측정
* 커버리지: Branch 기준 80%+ (핵심 유스케이스 90%+)
* Mutation testing(선택): 테스트 강도 점검
* 변경이 잦은 영역(핫스팟)은 테스트 우선

## 7. 정량 지표 가이드(가볍게)

### 함수
* 길이 ≤ 30줄, 파라미터 ≤ 4, 복잡도(Cyclomatic) ≤ 10

### 모듈
* Afferent(들어오는 의존)↑ 모듈은 안정적으로, Efferent(나가는 의존)↓ 유지
* 불가피하게 안정 모듈이 불안정 모듈에 의존하면 추상화 도입(DIP)

### PR 품질
* 플래그 인자 제거, 큰 DTO 축소, 사이클 없음, 테스트 동반 여부

## 8. PR 리뷰 체크리스트 (Yes/No)
* 변경 이유가 하나인가? (SRP)
* 외부 의존은 포트 뒤에 숨겼는가? (DIP)
* 큰 DTO/전역/플래그 인자 제거 또는 축소했는가?
* 트랜잭션/경쟁 조건을 DB에서 안전하게 처리했는가?
* 테스트(단위/계약/E2E)의 적절한 레벨이 같이 왔는가?
* 커버리지 공백이 분기 기준으로 남지 않았는가?

## 9. 개선 순서(리팩터링 로드맵)
  1. 핫스팟 탐지: 자주 고치는 파일/버그 많은 영역(로그/PR 기록)  
  1. 시임(Seam) 만들기: 포트 인터페이스 도입 → 외부 의존 격리  
  1. 유스케이스 단위 테스트 작성 → 빠른 피드백 루프 확보  
  1. 어댑터 분리: Supabase, 결제, 외부 API 구현을 모듈로 격리  
  1. 플래그/거대 DTO 제거 → 전략/작은 값객체  
  1. RLS/RPC 강화 → 클라이언트 로직 단순화  
  1. 중복 제거 & 네이밍 정리 → 응집 강화  

## 10. 짧은 예시 (FlutterFlow 호출 경계)
* FlutterFlow 위젯 → CartService.add(pid)만 호출
* CartService는 **포트 CartRepo**에 의존
* 실제 저장은 SupabaseCartRepo.add(user_id, pid)가 RPC cart_increment 호출
* RLS로 auth.uid() = user_id 보장 → 클라이언트는 단순/안전
