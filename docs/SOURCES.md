# 수집 소스 목록

| source | 상태 | 방식 | 주기 | 비고 |
|---|---|---|---|---|
| voice | 1단계 | 아이폰 액션버튼 → 음성메모 → 공유폴더 → Whisper | 매일 | 가장 신호 좋음 |
| claude_export | 1단계 | 설정 export JSON 파싱 | 주 1회 수동 | 내 발화 + 대화 요약 |
| obsidian | 2단계 | vault git diff | 매일 | 가장 쉬움 |
| chatgpt_export | 2단계 | conversations.json 파싱 (mapping 트리 → 선형화) | 월 1회 수동 | |
| youtube | 2단계 | Takeout(시청/검색) + Data API(좋아요/재생목록) | Takeout 월 1회, API 매일 | 시청기록은 노이즈 필터 필수 |
| search | 2단계 | 브라우저 히스토리 SQLite 복사 후 읽기 | 매일 | |
| git | 2단계 | 커밋 메시지·시각만 | 매일 | 회사 repo는 PRIVACY 필터 |
| quicknote | 2단계 | 회사 PC 단축키 한 줄 메모 → 파일 → Gitea push | 매일 | C# 소형 앱 |

새 수집기 추가 시 여기에 한 줄 등록.
