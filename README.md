# AI 트렌드 데일리 리포트

매일 아침 AI 뉴스를 자동 수집해 HTML 리포트로 정리하는 프로젝트입니다.

## 리포트 섹션

| 섹션 | 내용 |
|------|------|
| ⭐ 오늘의 하이라이트 | 가장 중요한 AI 소식 1~2개 |
| 🟣 Claude 소식 | Anthropic / Claude 업데이트 |
| 🟢 ChatGPT 소식 | OpenAI / ChatGPT 업데이트 |
| 🔵 Gemini 소식 | Google / Gemini 업데이트 |
| 🤖 AI 에이전트 트렌드 | 에이전트 관련 동향 |
| 📚 오늘의 AI 용어 | 초보자를 위한 용어 설명 |

## 파일 규칙

- 리포트 파일명: `YYYY-MM-DD.html`
- 저장 위치: 레포 루트 (`C:\daily report\`)
- 스케줄: 월~금 오전 8시 자동 생성, Windows 토스트 알림으로 열림

## 기술 스택

- AI: Claude (뉴스 검색 및 HTML 생성)
- 스케줄: Claude Code Scheduled Tasks
- 알림: PowerShell Windows Toast Notification
- 버전 관리: Git / GitHub (`jaeui2000-del/daily-roport`)
