---
layout: post
title:  "GoF Patterns"
date:   2025-08-31 21:34:00 +0900
categories: Patterns
---

GoF 패턴(Gang of Four, 1994)은 객체지향 설계에서 반복적으로 등장하는 해법 23가지를 정리한 “설계 어휘”예요. 목적은 응집도↑, 결합도↓, 변경 용이성↑ 입니다.

## 분류 한눈에
### 1) 생성(Creational)

* Factory Method: 생성 책임을 서브클래스에 위임

* Abstract Factory: 관련 객체군을 통째로 생성

* Builder: 복잡한 생성 과정을 단계로 분리

* Prototype: 복제(clone)로 생성 비용 절감

* Singleton: 인스턴스 1개 보장 (요즘은 DI로 대체 권장)

### 2) 구조(Structural)

* Adapter: 인터페이스 변환(끼워 맞추기)

* Bridge: 추상과 구현 분리(독립 진화)

* Composite: 트리 구조를 단일 객체처럼

* Decorator: 기능을 동적으로 덧입히기(래핑)

* Facade: 복잡 서브시스템에 단순한 입구

* Flyweight: 공유로 메모리 절약(가벼운 객체)

* Proxy: 대리자(지연 로딩, 캐시, 접근 제어)

### 3) 행위(Behavioral)

* Chain of Responsibility: 책임 연쇄(다음 핸들러로)

* Command: 요청을 객체로 캡슐화(undo/큐)

* Interpreter: 미니 언어/문법 해석

* Iterator: 순회 로직 분리

* Mediator: 상호 작용을 중재자에 모음

* Memento: 스냅샷(되돌리기)

* Observer: 이벤트 구독/발행

* State: 상태별 행위 캡슐화(상태 객체 전환)

* Strategy: 알고리즘 교체(동일 인터페이스)

* Template Method: 골격은 상위, 세부는 하위

* Visitor: 구조는 고정, 연산을 외부 방문자로 추가

## 실무에서 “자주 쓰는 10”과 언제 쓰나

* Strategy: 결제수단/정렬/가격정책처럼 “알고리즘 교체”

* State: 주문수명주기(장바구니→결제대기→확정…)처럼 “상태에 따라 동작 변화”

* Observer: 도메인 이벤트, 알림, 웹훅

* Decorator: 로깅/리트라이/캐시를 기존 서비스에 “덧입히기”

* Facade: 복잡한 서브시스템(결제 SDK, 서드파티 API) 단순화

* Adapter: 외부 API 모델 ↔ 내부 도메인 모델 변환

* Proxy: 지연로딩/캐시/권한 체크/레이트 리밋

* Composite: 카테고리 트리, UI 위젯 트리

* Factory/Abstract Factory: 환경/테넌트별 구현 선택

* Builder: 복잡한 DTO/메시지 조립(필수/옵션 분리)

## 헷갈리기 쉬운 것들 정리

* #### Adapter vs Facade

    - Adapter: 인터페이스를 바꿔 맞춤

    - Facade: 사용법을 단순화(내부는 그대로)

* #### Proxy vs Decorator

    - Proxy: 접근 전/대신 수행(원격, 캐시, 권한)

    - Decorator: 기능을 덧붙임(여러 개 중첩)

* #### Strategy vs State vs Template Method

    - Strategy: 런타임 알고리즘 교체

    - State: 상태 전이에 따라 내부 전략이 바뀜

    - Template: 상위 클래스가 골격 고정, 하위가 훅 구현

* #### Factory Method vs Abstract Factory vs Builder vs Prototype

    - FM: 한 종류 생성 책임 위임

    - AF: 관련 객체 묶음 생성

    - Builder: 단계적 조립

    - Prototype: 복제로 빠르게 생성

    - Singleton: 전역 상태화/테스트 어려움 → DI 컨테이너로 대체 권장

### 미니 예시 ① Strategy (결제 게이트웨이)
```ts
interface PayStrategy { pay(amount: number): Promise<void>; }
class Paypal implements PayStrategy { /* ... */ }
class Stripe implements PayStrategy { /* ... */ }

class Checkout {
  constructor(private strategy: PayStrategy) {}
  async confirm(amount: number) { await this.strategy.pay(amount); }
}
// 주입만 바꾸면 알고리즘 교체 끝
```

### 미니 예시 ② Decorator (로깅 덧입히기)
```ts
interface ProductRepo { get(id: string): Promise<Product>; }

class DbRepo implements ProductRepo { /* 실제 DB 조회 */ }

class LoggingRepo implements ProductRepo {
  constructor(private inner: ProductRepo) {}
  async get(id: string) {
    const t0 = Date.now();
    const res = await this.inner.get(id);
    console.log("get", id, Date.now()-t0, "ms");
    return res;
  }
}
```

## 어떻게 선택할까? (간단 규칙)

* 조건/분기 줄이고 싶다 → Strategy/State/Template

* 외부/레거시를 붙여야 한다 → Adapter/Facade

* 기능을 끼웠다 뺐다 → Decorator

* 지연/캐시/권한 → Proxy

* 트리 구조 → Composite

* 생성 복잡/환경별 구현 → Builder/Factory/Abstract Factory

## 모던 개발과의 연결

* DI/IoC가 보편화 → Factory/Singleton 필요성 ↓, Strategy/Decorator 적용 쉬움

* 함수형/컴포지션도 같은 목적(결합↓ 응집↑): 고차함수는 Strategy/Decorator와 유사

* DDD/이벤트 주도와 Observer/Command/Saga가 잘 맞물림

## 안티패턴 주의

* “패턴을 쓰기 위해 패턴을 쓰는 것” ⛔️

* 과도한 상속(Template/Factory 남용)보다 합성(Decorator/Strategy) 우선

* 싱글톤 남발 → 전역 상태, 테스트 곤란

---

쇼핑(카탈로그/장바구니) · 결제 · 재고 도메인을 기준으로 어디에 어떤 GoF 패턴이 적합한지 매핑표와, 바로 붙여 쓸 수 있는 TypeScript 코드 스켈레톤을 드릴게요.  
(Edge Functions/Deno·Node 어디서든 돌아가는 형태입니다. Supabase 연동 지점은 주석으로 표시했어요.)

## 1) 패턴 매핑표 (쇼핑/결제/재고)

|	도메인	|	문제/맥락	|	권장 패턴	|	이유/효과	|
|	---	|	---	|	---	|	---	|
|	카탈로그	|	카테고리·하위카테고리 트리	|	Composite	|	트리를 단일 객체처럼 취급(집계/탐색에 유리)	|
|	카탈로그 조회	|	DB→캐시→로그/메트릭 부착	|	Decorator	|	기존 Repo에 기능을 “덧입힘”(로깅/캐시)	|
|	외부 검색/번역 API 연동	|	내부 포트 ↔ 외부 SDK	|	Adapter	|	외부 인터페이스를 우리 포트에 맞춤	|
|	장바구니 금액 계산	|	할인/배송비/세금 교체	|	Strategy	|	알고리즘(정책) 교체·실험이 쉬움	|
|	주문 흐름	|	상태별 전이/행위 변경	|	State	|	PENDING/PAID/SHIPPED 등 상태 캡슐화	|
|	결제 오케스트레이션	|	사기탐지→승인→캡처→분개	|	Facade	|	복잡한 서브시스템을 단순 API로	|
|	결제 게이트웨이 다변화	|	Stripe/PayPal/Local PG	|	Factory / Abstract Factory	|	구현군 생성·선택을 분리	|
|	민감 API 접근	|	권한/캐시/지연 로딩	|	Proxy	|	접근 제어·캐시·회로차단 래퍼	|
|	도메인 이벤트	|	결제 성공→재고 예약 등	|	Observer	|	발행/구독으로 서비스 간 결합↓	|
|	주문 생성	|	필수/옵션 필드 분리 조립	|	Builder	|	유효성 보장하며 단계적 조립	|

(보너스, GoF 외) SAGA(보상): 주문↔결제↔재고 분산 트랜잭션 보정 시에 사용.

## 2) 코드 스켈레톤 (TypeScript)

아래 코드는 포트/어댑터 스타일로 서로 끼워 맞춰집니다.
파일/폴더 예시:

```css
src/
  catalog/ composite.ts, repo.ts
  cart/ strategy.ts
  order/ state.ts, builder.ts, events.ts
  payment/ facade.ts, adapter.ts, factory.ts
  inventory/ port.ts, subscriber.ts
  common/ proxy.ts, decorator.ts
```

### (1) Composite — 카테고리 트리

```ts
// src/catalog/composite.ts
export interface Node { name: string; count(): number; }

export class ProductLeaf implements Node {
  constructor(public name: string) {}
  count() { return 1; }
}
export class Category implements Node {
  private children: Node[] = [];
  constructor(public name: string) {}
  add(child: Node) { this.children.push(child); return this; }
  count() { return this.children.reduce((s, c) => s + c.count(), 0); }
}
```

### (2) Decorator — Repo에 로깅/캐시 덧입히기

```ts
// src/catalog/repo.ts
export interface Product { id: string; title: string; price: number; }
export interface ProductRepo { get(id: string): Promise<Product>; }

export class DbRepo implements ProductRepo {
  async get(id: string) {
    // TODO: Supabase select ... from products where id = $1
    return { id, title: "T", price: 100 };
  }
}

// src/common/decorator.ts
export class LoggingRepo implements ProductRepo {
  constructor(private inner: ProductRepo) {}
  async get(id: string) {
    const t0 = Date.now();
    try { return await this.inner.get(id); }
    finally { console.log("[repo.get]", id, Date.now() - t0, "ms"); }
  }
}
```

### (3) Adapter — 외부 결제 SDK ↔ 내부 포트

```ts
// src/payment/adapter.ts
// 외부 SDK(수정 불가)
class ExtPaySDK { async doPay(opts: { sum: number; currency: string }) { return { ok: true, id: "x1" }; } }

// 내부 표준 포트
export interface PayPort { pay(amount: number, currency: string): Promise<string>; }

export class ExtPayAdapter implements PayPort {
  constructor(private sdk = new ExtPaySDK()) {}
  async pay(amount: number, currency: string) {
    const res = await this.sdk.doPay({ sum: amount, currency });
    if (!res.ok) throw new Error("pay failed");
    return res.id;
  }
}
```

### (4) Strategy — 가격정책 교체

```ts
// src/cart/strategy.ts
import type { Product } from "../catalog/repo";
export type Item = { product: Product; qty: number };

export interface PricingStrategy { total(items: Item[]): number; }
export class DefaultPricing implements PricingStrategy {
  total(items: Item[]) { return items.reduce((s, i) => s + i.product.price * i.qty, 0); }
}
export class PercentOff implements PricingStrategy {
  constructor(private off: number) {}
  total(items: Item[]) { return new DefaultPricing().total(items) * (1 - this.off); }
}

// 사용: new Checkout(new PercentOff(0.1)).amount(items)
export class Checkout {
  constructor(private strategy: PricingStrategy) {}
  amount(items: Item[]) { return this.strategy.total(items); }
}
```

### (5) State — 주문 상태 전이

```ts
// src/order/state.ts
export interface OrderState {
  name: string;
  pay(): OrderState; ship(): OrderState; cancel(): OrderState;
}

export class Pending implements OrderState {
  name = "PENDING";
  pay() { return new Paid(); } ship() { throw new Error("pay first"); } cancel() { return new Cancelled(); }
}
export class Paid implements OrderState {
  name = "PAID";
  pay(){return this} ship(){return new Shipped()} cancel(){return new Refunded()}
}
export class Shipped implements OrderState { name="SHIPPED"; pay(){return this} ship(){return this} cancel(){throw new Error("no")} }
export class Cancelled implements OrderState { name="CANCELLED"; pay(){return this} ship(){return this} cancel(){return this} }
export class Refunded  implements OrderState { name="REFUNDED";  pay(){return this} ship(){return this} cancel(){return this} }

export class OrderAgg {
  constructor(public id: string, public state: OrderState = new Pending()) {}
  pay(){ this.state = this.state.pay(); }
  ship(){ this.state = this.state.ship(); }
  cancel(){ this.state = this.state.cancel(); }
}
```

### (6) Facade — 결제 흐름 단순화

```ts
// src/payment/facade.ts
import type { PayPort } from "./adapter";

class FraudCheck { async pass(userId: string, amount: number){ return true; } }
class Ledger { async post(entry: any){ /* TODO: Supabase RPC: post_ledger */ } }

export class PaymentFacade {
  constructor(private pay: PayPort, private fraud = new FraudCheck(), private ledger = new Ledger()) {}
  async payOrder(userId: string, orderId: string, amount: number) {
    if (!await this.fraud.pass(userId, amount)) throw new Error("fraud");
    const payId = await this.pay.pay(amount, "USD");
    await this.ledger.post({ userId, orderId, amount, payId });
    return { payId };
  }
}
```

### (7) Factory / Abstract Factory — 게이트웨이군 선택

```ts
// src/payment/factory.ts
import type { PayPort } from "./adapter";
import { ExtPayAdapter } from "./adapter";

export interface Refund { refund(payId: string): Promise<void>; }
class DummyRefund implements Refund { async refund(_: string) {} }

export interface PaymentsFactory { gateway(): PayPort; refund(): Refund; }

// 예: 지역/테넌트/환경변수에 따라 구현 선택
export class DefaultPaymentsFactory implements PaymentsFactory {
  gateway() { return new ExtPayAdapter(); }
  refund()  { return new DummyRefund(); }
}
```

### (8) Proxy — 권한/캐시/회로차단 래퍼

```ts
// src/common/proxy.ts
export interface SecretOrderPort { getSecret(orderId: string): Promise<string>; }

export class AuthProxy implements SecretOrderPort {
  constructor(private inner: SecretOrderPort, private hasRole: (r: string) => boolean) {}
  async getSecret(id: string) {
    if (!this.hasRole("admin")) throw new Error("forbidden");
    return this.inner.getSecret(id);
  }
}

// (선택) 캐시/서킷브레이커를 같은 자리에서 래핑 가능
```

### (9) Observer — 이벤트 발행/구독(결제→재고)

```ts
// src/order/events.ts
type Handler<T> = (p: T) => Promise<void> | void;
export class EventBus {
  private m = new Map<string, Handler<any>[]>();
  on<T>(evt: string, h: Handler<T>) { this.m.set(evt, [...(this.m.get(evt)||[]), h]); }
  async emit<T>(evt: string, p: T) { for (const h of (this.m.get(evt)||[])) await h(p); }
}
export const bus = new EventBus();

// src/inventory/port.ts
export interface InventoryPort { reserve(orderId: string, items: {sku:string;qty:number}[]): Promise<string>; release(resvId: string): Promise<void>; }

// src/inventory/subscriber.ts
import type { InventoryPort } from "./port";
import { bus } from "../order/events";

export function wireInventorySubscribers(inv: InventoryPort){
  bus.on<{orderId:string; items:{sku:string;qty:number}[]}>("order.paid", async ({orderId, items}) => {
    await inv.reserve(orderId, items); // TODO: Supabase RPC: inv_reserve
  });
}
```

### (10) Builder — 주문 조립

```ts
// src/order/builder.ts
type Item = { sku: string; qty: number; price: number };
export type Order = { id: string; userId: string; items: Item[]; note?: string; coupon?: string };

export class OrderBuilder {
  private id = "O-" + Math.random().toString(36).slice(2,8);
  private userId = ""; private items: Item[] = []; private note?: string; private coupon?: string;

  setUser(u: string){ this.userId = u; return this; }
  addItem(sku: string, qty: number, price: number){ this.items.push({sku, qty, price}); return this; }
  setNote(n: string){ this.note = n; return this; }
  useCoupon(c: string){ this.coupon = c; return this; }

  build(): Order {
    if (!this.userId || this.items.length === 0) throw new Error("missing fields");
    return { id: this.id, userId: this.userId, items: this.items, note: this.note, coupon: this.coupon };
  }
}
```

### (보너스) SAGA 오케스트레이터(GoF 외) — 보상 트랜잭션 자리

```ts
// src/app/saga.ts
type Step = { name: string; do: (ctx:any)=>Promise<void>; compensate: (ctx:any)=>Promise<void> };

export async function runSaga(steps: Step[], ctx: any){
  const done: Step[] = [];
  try {
    for (const s of steps) { await s.do(ctx); done.push(s); }
  } catch (e) {
    for (const s of done.reverse()) { try { await s.compensate(ctx); } catch {} }
    throw e;
  }
}

// 사용 예: Order→Authorize→Reserve
/*
await runSaga([
  { name:"CreateOrder", do: c=>rpc.order_place(c), compensate: c=>rpc.order_cancel(c) },
  { name:"Authorize",   do: c=>pg.authorize(c),    compensate: c=>pg.void(c)        },
  { name:"Reserve",     do: c=>inv.reserve(c),     compensate: c=>inv.release(c)    },
], ctx)
*/
```

## 3) 어떻게 붙여 쓰면 되나 (짧은 조립 예)

```ts
// src/app/app.ts
import { DbRepo } from "../catalog/repo";
import { LoggingRepo } from "../common/decorator";
import { DefaultPricing, Checkout } from "../cart/strategy";
import { OrderAgg } from "../order/state";
import { PaymentFacade } from "../payment/facade";
import { DefaultPaymentsFactory } from "../payment/factory";
import { bus } from "../order/events";
import { wireInventorySubscribers } from "../inventory/subscriber";

async function main(){
  // 카탈로그 Repo에 로깅 데코레이터 적용
  const products = new LoggingRepo(new DbRepo());

  // 가격 전략 선택(AB 테스트/테넌트별로 교체)
  const checkout = new Checkout(new DefaultPricing());

  // 결제 Facade 조립(게이트웨이 선택은 팩토리에게)
  const pay = new PaymentFacade(new DefaultPaymentsFactory().gateway());

  // 재고 구독자 연결
  wireInventorySubscribers({
    async reserve(orderId, items){ /* Supabase RPC inv_reserve */ return "resv-1"; },
    async release(resvId){ /* Supabase RPC inv_release */ }
  });

  // 주문→결제→이벤트(재고 예약) 흐름 예
  const order = new OrderAgg("O-1");
  const amount = checkout.amount([{ product: await products.get("P1"), qty: 2 }]);
  const { payId } = await pay.payOrder("U1", order.id, amount);
  await bus.emit("order.paid", { orderId: order.id, items: [{ sku:"P1", qty:2 }] });
  order.pay(); // 상태 전이
}
```

