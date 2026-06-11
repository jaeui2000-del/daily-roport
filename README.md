# 🤖 AI 트렌드 데일리 리포트

매일 아침 최신 AI 뉴스를 자동으로 수집하여 HTML 리포트로 정리해주는 자동화 프로젝트입니다.

## 📋 리포트 구성

| 섹션 | 내용 |
|------|------|
| ⭐ 오늘의 하이라이트 | 가장 중요한 AI 소식 1~2개 |
| 🟣 Claude 소식 | Anthropic / Claude 최신 업데이트 |
| 🟢 ChatGPT 소식 | OpenAI / ChatGPT 최신 업데이트 |
| 🔵 Gemini 소식 | Google / Gemini 최신 업데이트 |
| 🤖 AI 에이전트 트렌드 | 에이전트 관련 동향 |
| 📚 오늘의 AI 용어 | 초보자를 위한 AI 용어 설명 |

## ⏰ 실행 스케줄

- **실행 시간**: 월~금 오전 8시 자동 실행
- **알림 방식**: Windows 토스트 알림 (클릭 시 브라우저에서 자동 오픈)
- **저장 위치**: `C:\daily report\YYYY-MM-DD.html`

## 📁 파일 구조

```
daily-roport/
├── README.md
├── 2026-06-11.html
├── 2026-06-12.html
└── ...
```

## 🛠 기술 스택

- **AI**: Claude (Anthropic) — 뉴스 검색 및 리포트 생성
- **스케줄**: Claude Code Scheduled Tasks
- **알림**: Windows Toast Notification (PowerShell)
- **버전 관리**: Git / GitHub

---

> 이 프로젝트는 AI 초보자가 매일 최신 AI 트렌드를 쉽게 파악할 수 있도록 만들어졌습니다.
