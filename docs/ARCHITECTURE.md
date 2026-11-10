# 아키텍처

## 4계층

```
[수집]  collectors/*  → RawItem → data/raw/{source}/{YYYY-MM-DD}.jsonl  (append-only)
   ↓
[저장]  SQLite brain.db
        items     : raw 인덱스 (source, source_id, ts, text, meta)  + FTS5
        summaries : 하루 단위·대화 단위 요약
        memory    : 사실 / 관심사 / 진행 중인 일 / 반복 패턴  (2차 산물, 재생성 가능)
        links     : item ↔ memory, memory ↔ memory
   ↓
[처리]  process/run_nightly.py  (cron, 매일 새벽)
        1. normalize  : raw → items 적재, 중복 제거
        2. summarize  : 소스별 하루 요약 (LLM)
        3. extract    : 엔티티·주제·미해결 질문 추출 (LLM)
        4. memory     : 기존 메모리와 대조 → 갱신/신규/연결
        5. patterns   : 주 1회. 반복·추세 탐지
        6. render     : vault/daily/YYYY-MM-DD.md 브리핑 생성
   ↓
[인터페이스]  Obsidian (1단계) → 웹 대시보드 (2단계) → 챗 (3단계)
```

## 설계 결정의 이유

**원본/메모리 분리** — 모델이 좋아지면 과거 전체를 다시 해석할 수 있어야 한다.
메모리는 언제든 `memory/rebuild.py`로 원본에서 재생성 가능해야 한다.

**야간 배치** — 사용 패턴이 "퇴근 후 확인"이므로 실시간이 필요 없다.
복잡도를 10배 낮추고 LLM 비용을 예측 가능하게 만든다.

**SQLite 단일 파일** — 개인 규모(하루 수백 건)에 외부 DB는 과하다.
FTS5로 전문 검색, 임베딩은 numpy 파일 또는 sqlite-vec로 충분.

**Obsidian을 1단계 출력으로** — 이미 쓰는 도구이고 graph view가 공짜 시각화.
웹 UI는 "뭘 보고 싶은지" 명확해진 뒤에.

## RawItem 포맷

```python
@dataclass
class RawItem:
    source: str        # "voice" | "claude" | "chatgpt" | "youtube" | "search" | "git" | ...
    source_id: str     # 소스 내 고유 ID (중복 방지 키)
    ts: datetime       # 발생 시각 (UTC)
    text: str          # 본문. 비어 있으면 안 됨
    meta: dict         # 소스별 부가정보 (채널명, URL, 대화 제목 등)
```

## 메모리 타입

| type | 설명 | 예 |
|---|---|---|
| fact | 변하지 않는 사실 | "Synology NAS에 Gitea 운영" |
| interest | 관심사 + 강도 + 최근 활동일 | "스마트팜 자동화, 강도 높음, 11/08" |
| thread | 진행 중인 고민·프로젝트 + 상태 | "PMAC 외주 통합 방식, 미결" |
| pattern | 반복 행동·추세 | "월말에 이직 관련 검색 증가" |
| question | 미해결 질문 | "회사 PC 외부 접근 허용 범위?" |

메모리 한 줄에는 반드시 근거 item ID 목록이 붙는다. 근거 없는 메모리는 없다.

## 보안 경계

```
회사 PC ──(추상화 통과분만)──→ 개인 Gitea / 음성메모
                                      ↓
                              NAS (원본 + DB, 외부 노출 없음)
                                      ↓
                      process/llm.py ──(요약 단계에서 선별)──→ Anthropic API
```

외부 LLM으로 나가는 텍스트는 `process/llm.py`가 로그에 남긴다. 뭐가 나갔는지 항상 추적 가능해야 한다.
