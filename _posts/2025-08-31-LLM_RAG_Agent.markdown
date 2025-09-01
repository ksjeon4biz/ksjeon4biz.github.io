---
layout: post
title:  "LLM과 RAG, Agent 핵심부터 실무까지 한 번에"
date:   2025-08-31 23:55:00 +0900
categories: AI
---

## LLM(대규모 언어 모델)이란?

* 정의: 대량의 텍스트로 사전학습(Pretraining)된 Transformer 계열 모델. 다음 토큰 예측을 통해 문장 생성·요약·번역·질의응답 등 수행.

* 특징

* 지식은 파라미터 안에(학습 시점 기준). 최신/사내 데이터엔 약함.

* 문맥 창(컨텍스트 윈도우) 내에 넣어준 정보는 잘 활용.

* 할루시네이션(그럴듯한 오답 생성) 가능 → 외부 근거로 정박(grounding) 필요.

## RAG(Retrieval-Augmented Generation)란?

* 정의: 질문 시 **검색(검색/벡터 검색)**으로 관련 문서를 찾아 프롬프트 컨텍스트에 주입한 뒤, LLM이 답을 생성하는 아키텍처.

* 목표

1. 최신/사내 지식 접속 (모델 재학습 없이)

1. 할루시네이션 감소 (출처로 근거 제시)

1. 비용·속도 효율 (파라미터 업데이트 없이 확장)

## RAG 파이프라인 한눈에

```mathematica
사용자 질문
   │
   ├─▶ 1) 쿼리 확장/정규화(선택: Rewriting, HyDE 등)
   │
   ├─▶ 2) 검색(Retrieval)
   │      - 벡터(임베딩), 키워드(BM25), 하이브리드, 필터/ACL
   │
   ├─▶ 3) 리랭킹(Re-ranking)
   │      - Cross-Encoder/LLM으로 정밀 순위 조정
   │
   ├─▶ 4) 컨텍스트 구성(Context Packing)
   │      - 관련 스니펫 조합, 중복 제거, 길이 제한
   │
   └─▶ 5) 생성(Answer Synthesis)
          - 답변 + 인용(출처), 포맷 제약, 단계별 추론(비공개)
```

## 실무 설계 핵심(타협 지점 정리)

### 1) 문서 전처리(Chunking)

* 크기: 일반 텍스트 200–400 토큰, 10–20% overlap 권장.

  * 코드/표/법령은 더 작게, 서술형 보고서는 더 크게.

* 방식:

  * 의미 기반(제목·헤더·문단 분할) → 검색 품질↑

  * 하드 슬라이딩 윈도우 → 구현 단순·누락↓

* 메타데이터: 출처(URL/문서ID/버전/작성일/권한/언어/섹션) 꼭 저장.

### 2) 검색 전략

* Dense(벡터): 임베딩 코사인/내적. 의미 유사 검색.

* Sparse(BM25): 키워드·기호·숫자·약어 강함.

* 하이브리드: 두 점수 가중 합/학습 결합 → 현업 기본값 추천.

* 필터링: 테넌트/권한/날짜/언어 메타 필터 필수.

* 멀티호ップ: 1차로 개요 문서 찾고 → 그 안의 링크/섹션 2차 검색.

### 3) 리랭킹

* Top-50 검색 → 크로스 인코더/LLM 리랭커로 Top-5 추리기.

  * 긴 문서에 핵심문장 추출 → 재리랭크 조합이 효과적.

### 4) 컨텍스트 구성

* 중복 제거, 출처 다양성 확보(한 문서에 편향 금지), 길이 제한 내 패킹.

* 표/코드/목록은 원문 블록을 유지해 왜곡 방지.

### 5) 생성(프롬프트)

* 역할·형식·데드라인(언어/톤/목차/JSON 스키마) 명시.

* 인용(출처 링크/문서명+페이지/줄) 요구.

* 모르는 건 **“모른다/근거 없음”**으로 답하도록 규정.

### 최소 구현 예시(Python 의사코드)

```python
def rag_answer(question):
    # 1) 쿼리 전처리/확장(선택)
    q = rewrite(question)  # or q = question

    # 2) 검색: 하이브리드
    dense_hits = vector_search(q, topk=50)
    sparse_hits = bm25_search(q, topk=50)
    candidates = merge_scores(dense_hits, sparse_hits, alpha=0.6)

    # 3) 리랭킹
    reranked = cross_encoder_rerank(q, candidates)[:5]

    # 4) 컨텍스트 구성
    context = pack_snippets(reranked, max_tokens=1500)  # 중복 제거/메타 포함

    # 5) 생성(출처와 함께)
    prompt = f"""
    당신은 근거 기반 어시스턴트입니다.
    질문: {q}
    아래 자료만 근거로 한국어로 답변하세요. 모르면 모른다고 하세요.
    자료:
    {context}
    형식:
    - 핵심 답변 5줄 이내
    - 필요한 경우 목록/표
    - 인용: [문서명, 섹션] 형태로 표시
    """
    return llm_generate(prompt)
```

### 품질 높이는 팁(효과 큰 것 위주)

* 질문 재작성(Query Rewriting): 모模糊한 질문을 명시적 키워드+용어로 변환.

* HyDE: 가짜(가설) 답을 잠깐 생성해 그걸로 검색 → 리콜↑.

* Rewriting 다국어화: 질문 언어→영문→원문 언어 모두로 검색.

* 표/숫자 강화: 숫자/기호 토크나이징 보강, BM25 가중↑.

* 템플릿 가드: “출처 없는 문장 금지”, “날짜·단위 표기 규칙” 등 명시.

### 평가(Evaluation)와 관측성

* Retrieval: Recall@k, nDCG, MRR (정답 문서가 Top-k에 있나)

* Answer:

  * 정량: EM/F1/ROUGE(형태기반),

  * LLM-as-a-Judge: Faithfulness(근거 일치), Groundedness(출처 근거 비율)

  * 휴먼 평가: 최종 품질 보증(표본 추출)

* 관측성: 쿼리→검색→리랭크→생성 트레이스(Latency·Top-k·점수 분포·인용 실패율) 대시보드화.

### 보안/권한/거버넌스

* 문서별 ACL/테넌트 격리를 검색 단계에서 강제(사후 필터링 금지).

* PII/민감정보 마스킹 → 임베딩/로그에 남지 않게.

* 버전/감사: 어떤 버전 문서를 근거로 답했는지 기록.

* TTL/갱신: 인덱스 리프레시 주기, 캐시 무효화 정책.

### 성능·비용(지연 줄이는 법)

* 캐시: 쿼리→Top-k 후보, 리랭크 결과, 생성 결과까지 캐시 계층화.

* 스트리밍 응답: 헤더/요약 먼저, 세부는 뒤따라 전송.

* 샤딩/프리필터: 메타필터로 후보군 줄여 벡터 검색 비용↓.

* 지연 분할: 1차 빠른 답(Top-3) → 필요 시 “더 찾아보기” 버튼으로 확장.

### RAG vs 미세조정(Fine-tuning) — 언제 무엇을?

* RAG가 더 적합: 최신/사내 데이터, 근거 제시 필요, 빈번한 지식 변경.

* 미세조정이 더 적합: 스타일·포맷 습관화, 도메인 작업 절차(예: 버그 triage 규칙) 학습.

* 함께: 미세조정으로 응답 형식/도메인 용어를 익히고, 지식은 RAG로 공급.

### 프로덕션 레시피(요약)

1. __문서 파이프라인__: 수집→정규화→Chunking→임베딩→인덱스(벡터+BM25)

1. __질문 처리__: 재작성→하이브리드 검색→리랭크→컨텍스트 패킹

1. __생성__: 템플릿/포맷 가드, 출처 표기

1. __보안__: ACL 프리필터, PII 가드

1. __관측성/평가__: 리콜·Faithfulness·지연 대시보드

1. __운영__: 캐시·갱신 주기·비용 알람, 실패 시 폴백(“검색 결과 없음” 처리)

---

“제품/정책/FAQ”가 있는 쇼핑·결제·재고 도메인을 가정해, 바로 쓸 수 있는 RAG 초기 스펙을 한 번에 드릴게요. (Supabase Postgres + pgvector + Edge Functions 기준, 다른 스택도 쉽게 치환 가능합니다.)

## 1) 목표와 범위

* 목표: LLM이 최신/사내 지식을 활용해 정확하고 인용 가능한 답변을 생성

* 성과 기준: Retrieval Recall@5 ≥ 0.8, 답변 Faithfulness ≥ 0.9, p95 지연 ≤ 2.5s

* 스코프: 제품 카탈로그, 운영/반품/배송 정책, 고객지원 FAQ (다국어 포함)

## 2) 스키마 & 인덱스 (Supabase/pgvector)

```sql
-- 확장
create extension if not exists vector;
create extension if not exists pg_trgm;

-- 문서 메타
create table if not exists documents (
  id uuid primary key default gen_random_uuid(),
  source_url text,
  title text,
  section text,          -- 상위 섹션/카테고리
  lang text,             -- "ko","en","pl"... (다국어)
  tenant_id text,        -- 멀티테넌트/고객사 구분
  version int default 1,
  acl jsonb,             -- 접근 제어(roles, groups, users)
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- 청크 저장
-- NOTE: 임베딩 차원(d) 는 사용 모델에 맞게 설정 (예: 1536/3072 등)
create table if not exists chunks (
  id uuid primary key default gen_random_uuid(),
  doc_id uuid references documents(id) on delete cascade,
  chunk_index int,
  content text not null,
  heading text,          -- h1/h2 등
  lang text,
  tokens int,
  meta jsonb,            -- { "product_id":"P123", "policy":"returns" ... }
  embedding vector(1536),
  tsv tsvector,
  created_at timestamptz default now()
);

-- FTS + 벡터 인덱스
create index on chunks using ivfflat (embedding vector_cosine_ops) with (lists=100);
create index on chunks using gin (tsv);
create index on chunks using gin ((content) gin_trgm_ops);

-- FTS 채우기(간단: simple, 가능하면 언어별 tsconfig 사용)
create or replace function chunks_fill_tsv() returns trigger as $$
begin
  new.tsv := to_tsvector('simple', coalesce(new.heading,'') || ' ' || new.content);
  return new;
end$$ language plpgsql;

drop trigger if exists trg_chunks_tsv on chunks;
create trigger trg_chunks_tsv before insert or update of content, heading
  on chunks for each row execute procedure chunks_fill_tsv();
```

### 메타데이터 권장 필드

* tenant_id, lang, section, product_id, policy_type, effective_date, version, acl

* 이유: 검색 단계에서 **프리필터(권한/언어/테넌트/유효기간)**를 적용해야 안전합니다.

## 3) 청크/임베딩 파이프라인(수집·전처리)

### 청크 규칙

* 정책/가이드: 300–500 토큰, overlap 15%

* FAQ: Q/A 한 쌍 = 한 청크

* 제품: 상품 1개 + 주요 속성 블록(스펙 표는 원문 블록 유지)

### 파이썬 의사코드

```python
def ingest(doc):
    # 1) 정규화: 제목/헤더/표 추출, 언어감지, 메타 채우기
    chunks = semantic_split(doc.text, target_tokens=400, overlap=60)
    # 2) 임베딩
    vecs = embed([c.text for c in chunks])  # 모델: 멀티링궐 지원
    # 3) 저장
    for i, (c, v) in enumerate(zip(chunks, vecs)):
        upsert_document(doc.meta)
        insert_chunk(doc_id=doc.id, chunk_index=i, content=c.text,
                     heading=c.heading, lang=doc.lang, tokens=c.tokens,
                     meta=c.meta, embedding=v)
```

* 임베딩 모델: 다국어 지원(ko/pl/en 등). 차원(d)에 맞춰 vector(d) 지정.

## 4) 하이브리드 검색 + 리랭크 + 컨텍스트 패킹

### 4.1 검색 SQL(하이브리드: 벡터 + FTS)

```sql
-- 입력: :q (질의), :tenant, :lang[], :k, :alpha (0..1)
with q AS (
  select :q::text as text
),
vec as (
  select embedding(:q) as qvec      -- 임베딩 함수는 앱 계층에서 값 바인딩
),
cand as (
  -- 1) 벡터 top-N
  select c.id, c.doc_id, c.content, c.meta,
         1 - (c.embedding <=> (select qvec from vec)) as vscore,
         0.0::float as tscore
  from chunks c
  where c.lang = any(:lang) and c.doc_id in (
    select id from documents d
    where (d.tenant_id = :tenant or :tenant is null)
  )
  order by c.embedding <=> (select qvec from vec)
  limit 50
  union all
  -- 2) 키워드/FTS top-N
  select c.id, c.doc_id, c.content, c.meta,
         0.0::float as vscore,
         ts_rank(c.tsv, plainto_tsquery('simple', (select text from q))) as tscore
  from chunks c
  where c.lang = any(:lang)
    and c.tsv @@ plainto_tsquery('simple', (select text from q))
  order by tscore desc
  limit 50
),
scored as (
  select *, ( :alpha * vscore + (1-:alpha) * tscore ) as score
  from cand
)
select * from scored
order by score desc
limit :k * 10;   -- 후보는 넉넉히
```

### 4.2 리랭킹

* 후보 100 → Cross-Encoder/LLM 리랭커로 Top-k(예: 6~8) 선정

* 비용/지연이 부담되면: 질문+청크 첫 512토큰만 넣어 경량 리랭크

### 4.3 컨텍스트 패킹(프롬프트에 넣을 자료 만들기)

* 중복 제거(같은 doc_id 과다 편중 금지)

* 출처 다양성(FAQ/정책/제품 최소 2종)

* 토큰 한도 내에서 문단 경계 유지 (표/코드는 원문 블록 그대로)

## 5) 생성 템플릿(한국어, 인용 강제)

```text
역할: 당신은 근거 기반 어시스턴트입니다. 아래 자료만 근거로 답하세요.
언어: 한국어. 수치/날짜/단위는 원문을 따릅니다. 모르면 "근거가 없어 답할 수 없습니다."라고 말하세요.

[질문]
{{question}}

[자료]
{{#each contexts}}
- [{{this.source_title}} §{{this.section}}] {{this.snippet}}
  (doc_id={{this.doc_id}}, chunk={{this.chunk_index}})
{{/each}}

[요청]
1) 핵심 답변을 5줄 이내 요약
2) 필요시 목록/표 사용
3) 각 주장 뒤에 인용을 붙임: 예) ({{source_title}} §{{section}})
4) 정책/반품/배송 조건은 날짜·금액·지역을 구체적으로 표기
```

### 출력 포맷(예시)

* 본문 + 마지막에 인용 목록:

  - 예) (반품정책 §기간) (배송정책 §해외배송)

## 6) Edge Function(검색→리랭크→생성) 스켈레톤

```ts
// deno / edge
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";
import { embed, rerank, generate } from "./llm_clients.ts"; // 모델 클라이언트 추상화

export default async function handler(req: Request) {
  const { q, tenant, lang = ["ko","en"] } = await req.json();
  const supa = createClient(DENO_ENV.SUPABASE_URL, DENO_ENV.SUPABASE_SERVICE_ROLE);

  // 1) 임베딩 + 하이브리드 후보
  const qvec = await embed(q);
  const alpha = 0.6, k = 6;
  const candidates = await supa.rpc("hybrid_search", { q, qvec, tenant, lang, k, alpha }); // ← 위 SQL을 RPC로 만들면 깔끔

  // 2) 리랭크
  const reranked = await rerank(q, candidates); // top-6 정도

  // 3) 컨텍스트 패킹
  const contexts = pack(reranked, { maxTokens: 1600, diversifyBy: "doc_id" });

  // 4) 생성
  const answer = await generate({
    template: "grounded_ko",
    vars: { question: q, contexts }
  });

  return new Response(JSON.stringify({ answer, contexts }), { headers: { "content-type": "application/json" }});
}
```

## 7) 평가 & 대시보드(오프라인/온라인)

### 7.1 오프라인 세트(Excel/CSV)

```bash
id, question, gold_doc_ids, gold_spans, accept_answers
Q1, "반품 기간은?", "{docA,docB}", "문단#12", "14일,30일 정책 모두 허용"
...
```

* Retrieval: Recall@k, nDCG, MRR

* Answer:

  * 정량: EM/F1(키워드), Rouge-L

  * 심사: LLM-as-a-judge로 Faithfulness/Helpfulness (샘플 200건 수동 검증 병행)

### 7.2 온라인 로깅 테이블

```sql
create table if not exists rag_logs (
  qid uuid primary key default gen_random_uuid(),
  ts timestamptz default now(),
  question text,
  retrieved_ids uuid[],
  clicked_doc uuid,            -- 선택적(UX에서 클릭 수집)
  answer text,
  citations jsonb,
  latency_ms int,
  tenant_id text,
  user_id text
);
```

#### 지표 쿼리 예시

```sql
-- Recall@5 근사(오프라인 gold 테이블 join 가정)
select date_trunc('day', ts) d, avg( case when gold && retrieved_ids[1:5] then 1 else 0 end ) as recall5
from rag_logs l join gold_labels g on l.qid=g.qid
group by 1 order by 1;
```

#### 대시보드

* 검색 리콜, 인용 실패율, 평균/분위수 지연, API 오류율, 모델 비용(토큰) 추정

## 8) 운영 가이드(짧게)

* 인덱스 갱신: 문서 변경 시 Outbox → 재임베딩 큐 → 부분 업데이트

* 캐시: 쿼리→후보→리랭크→최종 응답 계층 캐시(+ TTL/Jitter)

* 권한: tenant_id/acl 필터는 검색 SQL에서 강제(사후 필터 금지)

* PII/민감정보: 임베딩 전 마스킹(전화/메일/카드BIN 등)

* 비용/지연: 하이브리드 후보 줄이기(프리필터), 리랭크/생성 스트리밍 응답

## 9) 체크리스트(배포 전)

 * 임베딩 차원 ↔ vector(d) 일치

*  하이브리드 검색 RPC에 권한 필터 포함

*  최대 토큰 초과 시 문단 경계 유지 패킹

*  인용 표기 강제 템플릿 적용

*  로그/대시보드 연결 및 알람(리콜 급락, 지연 급등)

 ---

 한 줄로 요약하면,

  **에이전트(Agent)**를 붙이면 LLM+RAG가 “지식 답변기”에서 목표 지향 실행기로 바뀝니다.
  스스로 계획→도구 호출→관찰→수정을 반복하며, RAG(사내 지식)·웹·DB·업무 API를 필요할 때 조합해 일을 끝냅니다.

아래에 무엇을 할 수 있는지, 대표 패턴, 쇼핑/결제/재고 예시, 그리고 바로 쓰는 코드 스켈레톤을 간단히 정리했어요.

## 1) Agent를 도입하면 가능한 일들

### 정보 정확도·최신성 업그레이드

* 동적 RAG: 질문을 재작성하고(쿼리 확장), 여러 번 검색·리랭크·재검색하며 다중 홉 자료를 모음.

* 소스 믹스: RAG(사내) + 웹 검색 + **DB 질의(SQL)**를 상황에 맞게 선택·결합.

* 자체 검증: 답을 만든 뒤 **자기점검(Reflection)**으로 출처 일치/수치 검증, 불확실하면 추가 검색.

### 워크플로 자동화

* 액션 실행: 주문 확인 → 환불 요청 생성 → 재고 예약 해제 → 고객 알림까지 엔드투엔드.

* 티켓/이메일/슬랙: 조건 충족 시 자동 생성·전달(휴먼 승인 게이트 포함).

* 요청 분류/라우팅: 문의를 청구/배송/반품 큐로 자동 분배, 필요한 데이터 조회 후 응답 초안 작성.

### 운영 지능화

* 도구 라우팅: “지금은 RAG로 충분 vs 웹 필요 vs SQL이 정답”을 비용·지연·신뢰도 기준으로 선택.

* 개인화/메모리: 고객/오퍼레이터 선호를 기억(언어·통화·반품 이력)해 대화·조치 최적화.

## 2) 대표 설계 패턴 (간단 지도)

* ReAct/Plan-Execute: 계획 → 도구 호출 → 관찰을 반복(최대 N스텝, 예: 6).

* MRKL/Toolformer: “도구 라우터”가 어떤 도구를 어떤 인자로 부를지 결정.

* Planner–Executor–Critic(Reflexion): 기획자/실행자/감사자(검토자)로 역할 분리.

* 싱글 에이전트 + 도구 ↔ 멀티 에이전트(오케스트레이터-워커): 규모·복잡도에 따라 선택.

#### 가드레일 필수
권한·스코프·쿼터, 입력 검증(JSON 스키마), 최대 단계/비용 한도, 프롬프트 인젝션 방어(허용 리스트/컨텍스트 샌드위치) 등을 반드시 둡니다.

## 3) 도메인 예시 (쇼핑/결제/재고)

### 예시 A) “폴란드 배송 가능? 비용·반품 기한?”

1. 계획: (i) 정책 KB 검색, (ii) 지역별 배송비 SQL 조회, (iii) 최신 요율 필요 시 웹.

1. 행동

  * RAG(정책/FAQ) → “해외배송 가능 국가/조건” 파편 확보

  * SQL: SELECT price FROM shipping_fee WHERE country='PL' AND weight<=X

  * 웹: 택배사 요율 페이지 확인(필요 시)

1. 검증/합성: 상충 시 재조회 → 답변을 표로 정리 + 인용(문서·행·URL).

### 예시 B) “주문 취소하고 환불해줘”

1. RAG: 환불 정책 확인(수수료/마감시간).

1. SQL: 주문 상태·결제 수단 확인 → 승인(Authorize)만이면 VOID, 이미 Capture면 Refund.

1. 결제 API 호출 → 재고 예약 해제(RPC) → 고객에게 결과 통지(이메일/푸시).

1. 실패 시 보상 트랜잭션(SAGA) 실행.

### 예시 C) “상품 URL을 주면 카탈로그에 등록”

1. 웹 스크래핑 → 스펙 추출 → 분류/속성 매핑(Strategy) → 이미지 업로드 → DB upsert → 번역 요청 큐잉.

## 4) 최소 코드 스켈레톤 (TypeScript, 단일 Agent)

  아이디어: RAG/SQL/WEB/ACT를 “도구(tool)”로 등록하고,
  LLM이 매 스텝마다 **{tool, args}**를 선택 → 실행 → 결과 관찰 → 종료 또는 다음 스텝.

```ts
// tools.ts — 도구 레지스트리
export type ToolResult = { ok: boolean; data?: any; error?: string; meta?: any };
export type Tool = (args: any, ctx: Ctx) => Promise<ToolResult>;
export type Ctx = { userId?: string; tenant?: string; budget: number; cites: any[] };

export const tools: Record<string, Tool> = {
  ragSearch: async ({ q, k = 6, lang = ["ko","en"] }, ctx) => {
    // 하이브리드 검색 RPC 호출 (pgvector+FTS)
    const res = await rpcHybridSearch({ q, k, lang, tenant: ctx.tenant });
    ctx.cites.push(...res.map((r: any) => ({ docId: r.doc_id, chunk: r.chunk_index })));
    return { ok: true, data: res };
  },
  sqlQuery: async ({ sql, params }, ctx) => {
    if (!sql.toLowerCase().startsWith("select")) return { ok:false, error:"readonly only" };
    const rows = await runReadOnlySQL(sql, params);
    return { ok: true, data: rows };
  },
  webFetch: async ({ url }, _ctx) => {
    const html = await fetchText(url);              // 안전한 도메인 화이트리스트
    const text = extractMainText(html);
    return { ok: true, data: text };
  },
  callPayment: async ({ action, payload }, _ctx) => {
    // action: "void" | "refund" | "authorize" | ...
    const res = await paymentApi(action, payload);  // 권한/검증/멱등 키 포함
    return { ok: res.success, data: res };
  },
  notifyUser: async ({ channel, message }, ctx) => {
    await sendNotification(ctx.userId!, channel, message);
    return { ok: true };
  },
};
```

```ts
// agent.ts — Planner/Executor 루프 (ReAct 스타일)
import { tools, Tool } from "./tools";
import { chat } from "./llm"; // 함수호출/툴선택이 가능한 LLM 클라이언트

type StepAction =
  | { type: "tool"; name: keyof typeof tools; args: any }
  | { type: "final"; answer: string };

export async function runAgent(question: string, init: Partial<Ctx> = {}) {
  const ctx: Ctx = { budget: 0.05 /*$*/, cites: [], ...init };
  const history: any[] = [{ role: "user", content: question }];

  for (let step = 1; step <= 6; step++) {
    const plan = await chat<StepAction>({
      system: `
        목표: 질문을 해결하라. 도구만 사용해 근거를 수집하고, 마지막에 인용을 포함해 한국어로 답변하라.
        규칙:
        - RAG로 시작, 부족하면 SQL/WEB을 사용
        - 도구 인수는 JSON으로 간결하게
        - 비용을 고려하여 불필요한 호출 금지
        - 충분한 근거가 모이면 type="final" 로 종료
      `,
      history,
      tools: Object.keys(tools),  // 모델이 선택 가능한 도구 이름
    });

    if (plan.type === "final") {
      return { answer: withCitations(plan.answer, ctx.cites), trace: history };
    }

    const tool: Tool = tools[plan.name];
    const out = await tool(plan.args, ctx);
    history.push({ role: "tool", name: plan.name, content: JSON.stringify(out).slice(0, 4000) });

    if (!out.ok) {
      // 간단한 회복: 에러를 모델에 전달해 수정 유도
      history.push({ role: "system", content: `도구 오류: ${out.error}` });
    }
  }

  return { answer: "최대 단계에 도달했습니다. 더 구체적 요청이 필요합니다.", trace: history };
}
```

#### 포인트

* RAG는 하나의 툴: Agent가 “필요하면 더 찾기/재질문/언어 바꾸기”를 스스로 결정.

* SQL은 Read-only부터 시작(쓰기/결제 같은 액션 툴은 별도 권한/승인).

* 가드레일: 도메인 화이트리스트, JSON 스키마 검증, 멱등 키, step/budget limit.

## 5) 운영 팁(바로 도움되는 것만)

* 성능: 1차 RAG 후 짧은 리랭크 → 결과 6~8개만 패킹 → 답 생성. 부족하면 2라운드.

* 비용: 결과 캐시(질문→후보→리랭크→최종답), 에이전트 최대 6스텝·시간/비용 상한.

* 신뢰: “출처 없는 문장 금지” 템플릿 + 강제 인용 + 수치/날짜 자체검증 루틴.

* 보안: 툴별 RBAC/테넌트 스코프, 비밀키는 서버측, 프롬프트 인젝션 방어(문서/웹 컨텐츠는 ‘참고 데이터’로만 취급, 시스템 지시 재설정 금지).

* 관측성: 스텝 로그(도구·인자·지연·오류), 성공률/평균스텝/오류 유형 대시보드.

## 6) 언제 Agent가 유리한가?

* 질문이 모호/다단계(정책+요율+재고 결합)

* 행동이 필요한 케이스(환불 생성, 레이블 발급, 알림 발송)

* 최신성/외부 근거가 중요(웹/사내 API 확인)

* 반대로, 단일 문서 요약/간단 Q&A는 순수 RAG가 더 빠르고 저렴합니다.

---

```sql
┌─────────┐                     ┌────────────────────────────────────────┐
│  User   │── 의도/질문 ───────▶  │                 Agent                  │
└─────────┘                     │ Plan → Tool-Use → Observe → Revise     │
                                │  ├─ Planner / Tool Router              │
                                │  ├─ Critic(자체 검증) / Context Guard    │
                                └──┬───────────────────────┬─────────────┘
                                   │                       │     |
                                   │ RAG 호출               │ LLM │
                                   ▼                       ▼     ▼
                        ┌─────────────────────┐      ┌────────────┐
                        │   RAG Subsystem     │      │    LLM     │
                        │  Retriever → Ranker │◀─────┤ (생성기)    │
                        │  → Context Packing  │  컨텍스트           │
                        └───────┬─────────────┘      └─────▲──────┘
                                │                          │ 초안
                                ▼                          │
                     ┌───────────────────┐                 │
                     │  Vector DB        │◀─ 임베딩/수집    ──┤
                     └────────┬──────────┘                 │
                              ▼                            │
                       Docs / FAQ / 정책                    │
                                                           │
Agent ──────────────▶  외부 도구: SQL/DB · Web · 업무 API(결제/재고)
   (선택적으로 호출)   (읽기/행동 실행, 승인·가드레일 적용)

Agent ── 검증/포맷/인용 ──▶ 최종 답변 ──▶ User
```

### 관계 요약

* LLM: 생성기. 들어온 프롬프트/컨텍스트로 답을 만들어냅니다.

* RAG: LLM에 **근거(사내 문서/정책/FAQ)**를 공급하는 검색·리랭크·컨텍스트 조립 서브시스템.

* Agent: 오케스트레이터. 질문에 맞춰 계획을 세우고 필요한 **도구(RAG/SQL/웹/API)**를 호출해 관찰→수정을 반복, 최종적으로 검증·인용을 붙여 사용자에게 답변/행동을 제공합니다.

