# 로드맵

원칙: **출력 먼저, 수집기는 체감 후.** 각 단계는 "실제로 매일 썼는가"로 완료 판정.

## 현재 단계: 1

## 1단계 — 아침 브리핑 하나가 돌아간다 (목표 1~2주)

- [ ] 프로젝트 골격, `collectors/base.py`, `RawItem`, SQLite 스키마
- [ ] 수집기 2개만:
  - [ ] `voice` — 아이폰 음성메모 → Whisper → RawItem (액션 버튼 단축어로 녹음, iCloud/공유폴더 경유)
  - [ ] `claude_export` — Claude 대화 export JSON 파싱
- [ ] `process/run_nightly.py` : normalize → summarize → render
- [ ] `vault/daily/YYYY-MM-DD.md` 브리핑 생성 (형식은 docs/BRIEFING.md)
- [ ] Mac에서 수동 실행으로 1주일 사용

완료 조건: 아침에 Obsidian 열어서 어제 브리핑을 7일 연속 읽었다.

## 2단계 — NAS 운영 + 수집기 확장 (목표 3~4주)

- [ ] Dockerfile, NAS cron 등록
- [ ] 1단계에서 "이게 없어서 아쉬웠다"고 느낀 순서로 수집기 추가. 후보:
  - `obsidian` — vault 변경분 (가장 쉬움)
  - `chatgpt_export` — conversations.json 파싱 (월 1회 수동 export)
  - `youtube` — Google Takeout 시청/검색 기록 + Data API (좋아요/재생목록)
  - `search` — 브라우저 히스토리 SQLite
  - `git` — 커밋 메시지만 (회사 repo는 추상화 필터 통과분)
  - `quicknote` — 회사 PC 단축키 한 줄 메모 (C# 소형 앱, 파일 append → Gitea)
- [ ] 메모리 계층 (`memory/`) 도입: fact / interest / thread / question
- [ ] 주간 리포트

## 3단계 — 패턴과 연결 (한 달치 데이터 이후)

- [ ] 임베딩 + 유사 생각 연결 ("3개월 전 같은 고민")
- [ ] 패턴 탐지 (주기성, 추세)
- [ ] 질문 챗 (CLI → 웹)
- [ ] 시각화: 주제 타임라인, 관심사 지도

## 보류 / 하지 않기로 한 것

- 상시 녹음 웨어러블 (회사 보안, 동료 동의 문제)
- 클립보드 전체 수집 (비밀번호 혼입 위험, 필터 설계 전까지)
- 실시간 처리
