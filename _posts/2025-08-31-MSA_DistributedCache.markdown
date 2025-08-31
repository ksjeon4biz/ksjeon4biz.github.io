---
layout: post
title:  "MSA에서 분산캐시 적용 방안"
date:   2025-08-31 17:57:00 +0900
categories: MSA
---

MSA에서 분산 캐시는 “느린 것(DB/외부 API/복잡 계산)” 앞에 놓는 완충 장치입니다. 각 마이크로서비스가 자기 데이터의 읽기 경로를 빠르게 하면서도, 서비스 간 결합은 낮게 유지하는 게 핵심이에요. 아래에 어디에, 어떤 패턴으로, 어떻게 안전하게 쓰는지 딱 정리했습니다.

## 어디에 캐시를 두나

* __에지/게이트웨이 캐시__: API Gateway/NGINX에서 공개 GET 응답을 짧게 캐시.

* __서비스별 분산 캐시(핵심)__: Redis/Memcached 같은 공유 캐시 클러스터를 각 서비스가 사용.

    * 필요하면 Near Cache(인스턴스 로컬 메모리) + 백엔드 분산 캐시 2단 구조.

* (지양) 여러 서비스가 같은 키를 계약처럼 공유하는 방식은 결합↑ → 피하고, “각 서비스가 자기 리소스만 캐시”가 원칙.

## 필수 패턴 6가지

### 1. Cache-aside (가장 일반적)

    * 읽기: cache.get → miss면 DB/외부 호출 → 값 저장(TTL) → 반환

    * 쓰기: DB 업데이트 후 해당 키 무효화/갱신

### 2. Read-through

    * 캐시가 miss 시 원천을 직접 읽어 채움(라이브러리/프락시가 지원)

### 3. Write-through / Write-behind

    * Write-through: 캐시에 쓰면 동시에 DB 반영

    * Write-behind: 캐시에 먼저 쓰고 비동기로 DB 반영(지연 허용될 때만)

### 4. Refresh-ahead

    * 만료 임박(soft TTL) 시 백그라운드 리프레시, 사용자는 항상 빠른 응답

### 5. Near Cache

    * 인스턴스 로컬 메모리에 아주 짧게 보관, 불일치 위험은 Pub/Sub로 완화

### 6. Negative Caching

    * “없음(404/빈 결과)”도 잠깐 캐시해서 DB 폭주 방지

## 무효화/일관성(어려운 부분) 처리

* __TTL+Jitter__: TTL에 무작위 편차(±10%)를 줘서 동시만료(스파이크) 방지

* __쓰기 시 즉시 무효화__: 업데이트/삭제가 끝나면 해당 키 삭제

* __Pub/Sub 브로드캐스트__: 변경 시 product:123 키를 구독자에게 무효화 이벤트로 알림

* __버전드 키__: v3:product:123:ko-KR처럼 버전을 키에 포함 → 배포 시 버전만 올려 대량 무효화

* __CDC 기반__: DB 변경 로그(예: WAL/CDC) 이벤트로 관련 키를 계산해 무효화

## 캐시 폭주(Stampede) 방지

* __분산 락(single-flight)__: 최초 1개 요청만 원천 호출, 나머지는 잠깐 대기

* __stale-while-revalidate__: hard TTL과 별개로 soft TTL을 두고, 살짝 낡은 데이터라도 즉시 제공 + 백그라운드 갱신

* __백오프/서킷브레이커__: 원천 장애 시 캐시 TTL을 일시 연장(브라운아웃)

## 키/TTL 설계 가이드

* __키 규칙__: {버전}:{리소스}:{id}:{locale}:{tenant}

    * 예) v5:product:123:ko-KR:t_acme

* TTL 힌트

    * 거의 안 바뀜(카탈로그 메타): 5~30분

    * 자주 바뀜(가격/재고): 3~60초 + Pub/Sub 무효화

    * 사용자별·권한 데이터: 30~300초(리스크에 따라 축소)

## 마이크로서비스 특화 활용처

* __읽기 모델(CQRS)__: 조회 전용 API는 캐시/머티리얼라이즈드 뷰로 서빙

* __집계/추천/랭킹__: 배치/스트림으로 계산 → 결과만 캐시

* __권한/토큰 인트로스펙션__: 짧은 TTL로 캐시

* __레이트 리밋/세마포어/분산락__: Redis 카운터/락으로 제어

* __Idempotency Key 저장__: 중복 요청 방지

* __사이드카/컴포지터__: API 합성 서비스가 부분 결과 캐시해 N+1 호출 축소

## 운영 체크리스트

* __소유권__: 각 서비스는 자기 데이터만 캐시. 캐시를 서비스 간 통신 버스로 쓰지 말 것.

* __관측성__: hit/miss, p99, 키수, 메모리 사용, eviction, 오류율 모니터링

* __장애 모드__: 캐시 다운 시 DB 보호(서킷브레이커, 임시 TTL 증가, 부분 비활성화)

* __보안/멀티테넌시__: 키 prefix로 테넌트 격리, 민감데이터는 캐시에 저장 자체를 재검토

* __용량/정책__: allkeys-lru/volatile-ttl 등 정책 선택, 큰 값은 압축 고려

* __리전/가용영역__: AZ별 로컬 캐시 + 크로스-AZ 복제(읽기 지연/전송 비용 고려)

### 예시 코드 (Python, Redis, Cache-aside + 락 + SWR)

```python
import json, time
import redis

r = redis.Redis.from_url("redis://cache-cluster:6379/0")

HARD_TTL = 300   # 절대 만료
SOFT_TTL = 30    # 이 시간이 지나면 백그라운드 갱신
LOCK_TTL = 10

def _key(product_id, locale="ko-KR", ver="v5", tenant="t_acme"):
    return f"{ver}:product:{product_id}:{locale}:{tenant}"

def get_product(product_id):
    key = _key(product_id)
    raw = r.get(key)
    now = int(time.time())
    if raw:
        data = json.loads(raw)
        # SWR: soft_ttl 지나면 백그라운드 리프레시 트리거
        if data["soft_exp"] <= now:
            if r.setnx(key+":lock", "1"):
                r.expire(key+":lock", LOCK_TTL)
                # 비동기 워커 큐에 새로고침 enqueue (여기선 생략)
        return data["payload"]

    # Stampede 방지: 분산 락
    if r.setnx(key+":lock", "1"):
        r.expire(key+":lock", LOCK_TTL)
        payload = fetch_from_db(product_id)      # ← 원천 호출
        cache_val = json.dumps({
            "payload": payload,
            "soft_exp": now + SOFT_TTL
        })
        r.setex(key, HARD_TTL, cache_val)
        r.delete(key+":lock")
        return payload
    else:
        # 동시 호출은 짧게 대기 후 재시도 또는 약간 stale 허용 정책
        time.sleep(0.05)
        return get_product(product_id)
```

* 요점: miss 시 한 요청만 DB를 두드리고, 응답은 SWR로 부드럽게 갱신.

### 빠른 적용 순서

1. 캐시 대상 선정: 고비용/고빈도 조회부터 (카탈로그, 권한, 설정, 합성 응답)

1. 키/TTL/버전 규칙 확정, 히트율 목표(예: 80%+) 설정

1. 무효화 전략(TTL + Pub/Sub + 배포 버전) 마련

1. stampede 방지(락/SWR)와 장애 모드(브라운아웃) 구현

1. 대시보드/알람 구성 → 목표 미달 시 원인(키 폭증/TTL 과도/무효화 과다) 튜닝