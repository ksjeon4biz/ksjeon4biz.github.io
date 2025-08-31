---
layout: post
title:  "SAGA 패턴의 보상 트랜잭션"
date:   2025-08-31 16:36:00 +0900
categories: SAGA
---

좋아요! SAGA 패턴에서 **보상 트랜잭션(Compensation)**이 어떻게 동작하는지, 실무에서 바로 쓸 수 있는 예시로 설명할게요. (주문–결제–재고–배송 시나리오, 오케스트레이션/코레오그래피 두 방식, 보상 매핑표, 의사코드 포함)

## 1) 한 줄 정의

분산 트랜잭션에서 전체 단계를 한 번에 ACID로 묶을 수 없으니, 앞으로 진행(Forward) → 실패 시 역순으로 보상(Undo/보정) 하는 패턴이 SAGA입니다.
보상 트랜잭션은 “정확히 되돌리기”가 아니라 업무적으로 상응하는 보정(예: 환불, 예약 해제)을 의미해요.

## 2) 예시 시나리오: 주문 처리
### 단계(Forward)

1. __주문 생성(Order Service):__ PENDING 상태로 생성

1. __결제 승인(Authorize)(Payment):__ 금액만 묶음(아직 Capture 안 함)

1. __재고 예약(Inventory):__ sku/qty를 예약 상태로 차감

1. __배송 라벨 생성(Shipping):__ 운송장/라벨 발급

1. __주문 확정(CAPTURE):__ 결제 캡처 후 주문을 CONFIRMED

### 보상(Compensation, 실패 지점별 역순 실행)

* 배송 라벨 생성 실패 → (보상 없음, 이전 단계로 롤백 불필요)

* 재고 예약 실패 → 결제 승인 취소(VOID) → 주문 취소

* 결제 캡처 이후 실패 → 환불(Refund) → 재고/배송 취소 → 주문 취소

* 재고 예약까지 됐는데 배송 라벨 생성 실패 → 재고 예약 해제 → 결제 승인 취소 → 주문 취소

### “Forward ↔ 보상” 매핑표

|	Forward(정상 단계)	|	Compensation(보상 단계)	|	비고	|
|	---	|	---	|	---	|
|	주문 PENDING 생성	|	주문 CANCELLED	|	최후 보상	|
|	결제 Authorize	|	Void(승인 취소)	|	Capture 전	|
|	결제 Capture	|	Refund	|	이미 돈이 움직였으면 환불	|
|	재고 Reserve	|	Release(예약 해제)	|	수량 되돌림	|
|	배송 라벨 생성	|	라벨 Cancel	|	운송장 무효화	|


* 포인트: 가능하면 2단계 결제(Authorize→Capture)로 설계하면 보상이 쉬워집니다.

## 3) 두 가지 스타일
### A. 오케스트레이션(중앙 조정자)

중앙 Saga Orchestrator가 각 서비스를 호출하고, 실패 시 역순 보상을 지시합니다.

```ts
// 의사코드 (TypeScript 느낌)
type Step = {
  name: string;
  do: (ctx) => Promise<Partial<Ctx>>;
  compensate: (ctx) => Promise<void>;
};

const steps: Step[] = [
  { name: "CreateOrder",
    do: async (c) => ({ orderId: await order.create(c.userId) }),
    compensate: async (c) => order.cancel(c.orderId) },
  { name: "AuthorizePayment",
    do: async (c) => ({ authId: await pay.authorize(c.orderId, c.amount, c.idemKey) }),
    compensate: async (c) => pay.void(c.authId, c.idemKey) },
  { name: "ReserveStock",
    do: async (c) => ({ resvId: await inv.reserve(c.orderId, c.items) }),
    compensate: async (c) => inv.release(c.resvId) },
  { name: "CreateLabel",
    do: async (c) => ({ labelId: await ship.createLabel(c.orderId, c.addr) }),
    compensate: async (c) => ship.cancelLabel(c.labelId) },
  { name: "CapturePayment",
    do: async (c) => ({ payId: await pay.capture(c.authId) }),
    compensate: async (c) => pay.refund(c.payId) },
];

async function runSaga(ctx) {
  const done: Step[] = [];
  for (const s of steps) {
    try {
      Object.assign(ctx, await s.do(ctx));
      done.push(s);
    } catch (e) {
      // 역순 보상
      for (const b of done.reverse()) {
        try { await b.compensate(ctx); } catch (ce) { /* 재시도/수동처리 큐 */ }
      }
      throw e;
    }
  }
  return ctx;
}
```

__장점:__ 흐름이 명확, 실패 시 역보상이 쉬움
__주의:__ 오케스트레이터의 신뢰성/내구성(상태 저장, 재시도, 멱등성)을 확보해야 함

## B. 코레오그래피(이벤트 발행/구독)

각 서비스가 이벤트를 구독해서 다음 단계로 스스로 진행하고, 실패 시 자신의 보상 이벤트를 발행합니다.

* OrderCreated → Payment가 PaymentAuthorized 발행

* PaymentAuthorized → Inventory가 StockReserved 발행

* StockReserved → Shipping이 LabelCreated 발행

* 중간 실패 시 예: StockReserveFailed → Payment가 PaymentVoided 발행, Order는 OrderCancelled

__장점:__ 느슨한 결합, 확장 쉬움
__주의:__ 흐름 파악이 분산됨(관찰성/추적성 확보 필요: 코릴레이션 ID, 이벤트 스키마, 사가 로그)

## 4) 멱등성·재시도·오류 처리 핵심

* __Idempotency-Key__: POST /payments/authorize 같은 생성/변경 요청에 키를 부여해 중복 호출 방지

* __Outbox 패턴__: “로컬 트랜잭션 + 이벤트 기록”을 하나로 커밋, 별 프로세스가 이벤트 브로커로 내보냄 → 메시지 유실 방지

* __사가 로그__: saga_instance(id, state, last_step, context, updated_at)를 저장해, 재시작/재시도 가능

* __보상 실패__: 재시도 정책 + 수동 개입 큐(Dead-letter) 준비

* __순서__: 보상은 반드시 역순(LIFO)

* __시간 제한__: 각 단계/보상에 타임아웃과 서킷브레이커 설정

## 5) Supabase/FlutterFlow 맥락으로 옮기기

* __오케스트레이터__: Supabase Edge Function(Deno)에서 SAGA 실행 상태를 saga_instance 테이블에 기록

* __도메인 단계__: 각 서비스는 **RPC(원자 함수)**로 구현 (재고 예약/해제, 결제 승인/취소, 라벨 생성/취소)

* __RLS__: 사용자별 리소스 보호, 오케스트레이터용 서비스 롤 분리

* __멱등성__: Idempotency-Key를 헤더로 받아 각 RPC가 키별 중복 처리 방지 테이블을 확인

* __이벤트__: Outbox 테이블 + Supabase Realtime/작업 큐(크론/워커)로 발행

### 예: 재고 단계(RPC 예시, 간단화)

```sql
-- 예약
create or replace function inv_reserve(_order uuid, _sku text, _qty int, _key text)
returns uuid language plpgsql security definer as $$
declare _resv uuid := gen_random_uuid();
begin
  -- 멱등 처리
  if exists (select 1 from idem_keys where key=_key and op='inv_reserve') then
    return (select resv_id from idem_keys where key=_key and op='inv_reserve');
  end if;

  update stock set reserved = reserved + _qty, available = available - _qty
  where sku=_sku and available >= _qty;

  if not found then
    raise exception 'INSUFFICIENT_STOCK';
  end if;

  insert into reservations(resv_id, order_id, sku, qty) values (_resv, _order, _sku, _qty);
  insert into idem_keys(key, op, resv_id) values (_key, 'inv_reserve', _resv);
  return _resv;
end $$;

-- 보상(예약 해제)
create or replace function inv_release(_resv uuid)
returns void language plpgsql security definer as $$
declare _r record;
begin
  select * into _r from reservations where resv_id=_resv for update;
  if not found then return; end if; -- 멱등 보장
  update stock set reserved = reserved - _r.qty, available = available + _r.qty where sku=_r.sku;
  delete from reservations where resv_id=_resv;
end $$;
```

* 결제는 Authorize/VOID 중심, 이미 Capture 했다면 Refund로 보상.

## 6) 무엇이 “좋은 보상 트랜잭션”인가?

* 의미가 보전: 사용자가 체감하는 결과가 합리적(돈 환불, 수량 원상복구)

* 멱등/재시도 안전: 네트워크 오류에도 중복 부작용 없음

* 감사 가능: 누가, 언제, 무엇을, 왜 보상했는지 로그로 남김

* 부분 보정 허용: 외부 시스템 때문에 완벽히 “이전과 동일”은 불가할 수 있음(예: 환율, 수수료). 정책으로 정의

## 7) 빠른 체크리스트

* 각 Forward 단계마다 명시적 보상 단계가 정의되어 있는가?

* 보상은 역순으로, 멱등/재시도 가능한가?

* 오케스트레이션이면 사가 로그/상태머신이 있는가? 코레오그래피면 코릴레이션 ID/추적이 확보됐는가?

* Outbox/Idempotency-Key를 적용했는가?

* 결제는 Authorize→Capture로 2단계인가(보상 용이)?

* 수동 개입이 필요한 실패를 데드레터 큐로 모으는가?
