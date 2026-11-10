# second-brain — 개인 어시스턴트 / 생각 수집 시스템

흩어진 생각(음성메모, AI 대화, 유튜브, 검색, 커밋 등)을 자동 수집하고,
야간 배치로 요약·연결해서 "어제 내가 뭘 생각했나"를 아침에 보여주는 시스템.
장기 목표: 내 외부 흔적을 나보다 꼼꼼히 기억하고 연결하는 어시스턴트.

상세 설계는 `docs/` 참고. 이 파일은 작업 규칙만 담는다.

## 핵심 원칙 (반드시 지킬 것)

1. **원본은 절대 수정·삭제하지 않는다.** `data/raw/`는 append-only. 메모리·요약은 원본에서 재생성 가능한 2차 산물.
2. **수집(상시)과 해석(야간 1회)을 분리한다.** 수집기는 저장만 하고 LLM을 호출하지 않는다.
3. **외부 LLM에 보내는 데이터는 `process/` 단계에서만, 선별해서.** 수집기에서 API 호출 금지.
4. **회사 데이터는 추상화 규칙을 통과한 것만 저장.** 코드·고객사명·장비 스펙 금지. 판단·교훈·고민만. (`docs/PRIVACY.md`)
5. **평가하지 않는다.** 이 시스템은 기록과 연결만 한다. "어제 유튜브 3시간" 같은 자책 유도 출력 금지.
6. **출력 먼저, 수집기는 나중에.** 매일 쓰는 아침 브리핑이 먼저 돌아가야 한다. 수집기 추가는 "이 데이터가 없어서 아쉬웠다"는 체감 이후.

## 스택

- Python 3.12+, `uv`로 패키지 관리
- 저장: `data/raw/` (markdown/JSON 원본), SQLite (`data/brain.db`: 구조화 + FTS5), 임베딩은 SQLite 확장 또는 로컬 파일 — 외부 벡터DB 도입 금지(1단계)
- LLM: Anthropic API (처리 단계), 로컬 Whisper (음성→텍스트)
- 실행 환경: Mac(개발) → Synology NAS Docker(운영). 모든 수집기는 Docker에서 돌 수 있어야 함
- 출력: markdown 파일 → Obsidian vault(`vault/`)에 떨어뜨림. 웹 UI는 2단계 이후
- 스케줄: cron (NAS). 야간 배치 `process/run_nightly.py` 하나로 묶는다

## 디렉토리

```
collectors/     소스별 수집기. 하나의 소스 = 하나의 파일. 공통 인터페이스 collectors/base.py
process/        야간 배치: 정규화 → 요약 → 태깅 → 메모리 갱신 → 브리핑 생성
memory/         메모리 스키마, 읽기/쓰기, 재생성 스크립트
output/         브리핑·리포트 렌더러 (markdown)
data/raw/       원본 (append-only, git 추적 안 함)
data/brain.db   SQLite (git 추적 안 함)
vault/          Obsidian vault (별도 git 또는 Obsidian Sync)
docs/           설계 문서
tests/
```

## 수집기 규칙

- `collectors/base.py`의 `Collector` 를 상속. `fetch() -> list[RawItem]` 만 구현
- `RawItem` 공통 포맷: `source, source_id, ts, text, meta: dict`. 모든 소스가 이 포맷으로 떨어진다
- 중복 방지는 `(source, source_id)` 유니크 키. 수집기가 직접 중복 처리하지 않는다
- 수집기는 네트워크 실패 시 예외를 던지지 않고 빈 리스트 + 로그. 다른 수집기를 막으면 안 된다
- 새 수집기는 `docs/SOURCES.md`에 한 줄 등록

## 코드 규칙

- 타입 힌트 필수, `ruff` 통과
- 설정은 `.env` + `config.py`. 하드코딩된 경로·키 금지
- 로그는 `logging`, print 금지
- LLM 호출은 `process/llm.py` 하나를 통해서만. 프롬프트는 `process/prompts/*.md`에 파일로
- 테스트는 실제 LLM 호출 없이 돌아야 함 (fixture 사용)

## 작업 방식

- 작업 시작 전 `docs/ROADMAP.md`에서 현재 단계 확인. 단계를 건너뛰는 기능 요청이 오면 먼저 확인할 것
- 설계가 바뀌면 `docs/DECISIONS.md`에 한 줄 추가 (날짜, 결정, 이유)
- 한 번에 한 수집기 / 한 처리 단계만. 여러 소스를 동시에 만들지 않는다
- 커밋 메시지는 한국어, 한 줄 요약 + 필요시 본문

## 하지 말 것

- 수집기 안에서 LLM 호출
- `data/raw/` 수정
- 실시간 처리 파이프라인 도입 (야간 배치로 충분)
- 1단계에서 웹 프레임워크, 벡터DB, 메시지큐 도입
- 회사 코드/문서 내용을 그대로 저장하는 수집기
